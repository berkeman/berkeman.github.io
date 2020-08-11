---
layout: post
title:  "Parallel Image Processing using the Accelerator:  The Basics"
date:   2019-06-28 00:00:00
categories: processing
---

Updated 2020-08-11


The Accelerator is designed for fast and reproducible data processing.
Typical application data is composed of text strings and numbers, but
the Accelerator can work as efficiently with any kind of binary data,
such as images or sound files.  In this post we will have a look at
batch processing of large quantities of images.  The focus will be on

  - parallel processing, i.e. how to make most use of the computer
    hardware to save execution time, and

  - reproducibility, i.e. how to associate results (output) to
    specific input data and source code.
	
We will use a simple example to show *how* to make best use of the
Accelerator's capabilities.  More advanced examples, such as neural
network inference on image data, will be discussed in an upcoming
post.





# Parallel Image Processing Example

A simple example is that of creating image thumbnails (i.e. downscaled
copies) of a large set of (larger) images.  The figure below depicts
conversion from a 4k video frame down to a size that is more
reasonable for a neural network to operate on

<p align="center"><img src="{{ site.url }}/assets/image_rescale.svg"> </p>

Throughout this post, we'll assume that the number of parallel
processes is three (3) to keep things simple.  In an actual setup,
this number is equal or close to the number of available CPU cores on
the computer, which may be significantly higher.

We will approach the example from two directions,

1. by keeping things simple and writing a minimal parallel program to solve the task, and
2. by using more Accelerator features to create a solution that is
   more flexible and extendable.

The second approach will scale much better with an increased
complexity of problems to solve.  But let us start with the simple
straight-forward solution.





# Straightforward Parallel Program

The downscaling program receives a list of images to process as input,
and outputs a set of thumbnail image files.  To make most use of the
available hardware, the program will run in parallel on several CPU
cores.  Each parallel process will work on a unique slice of the input
image file list.  For each filename in the list, the process will read
the corresponding input image, downscale, and write the output
thumbnail image, see the figure below.

<p align="center"><img src="{{ site.url }}/assets/image_files.svg"> </p>

Here is the complete source code for the program.  We use the
PIL/Pillow library for image processing.

```python
from PIL import Image

options=dict(files=[], size=(100, 100))

def analysis(sliceno, slices, job):
    files = options.files[sliceno::slices]
    for fn in files:
        im = Image.open(fn)
        im.thumbnail(options.size)
	filename = fn + '.thumbnail', 'JPEG'
        im.save(filename)
	job.register_file(filename)
```

The function `analysis()` will be forked and executed in `slices`
parallel processes (where `slices` is set in the Accelerator's
configuration file).  Each process receives a unique number,
`sliceno`, varying between zero and the number of slices. Input
options are the list of image file names and the shape of the output
thumbnail image.  (We use `job.register_file()` to record that a file
was created in the job, this is convenient but not necessary in any
way.)

In order to execute the program, we need to write a small build script
containing the build rules

```python
def main(urd):
    urd.build('thumbnailer', files=['file0.jpg', ...], size=(640, 338))
```

This program will create approximately **140 4k-to-640x338-thumbnails per second** on a
modern laptop with four cores.




# Using More of the Accelerator's Features

The program in the previous section provides a simple but efficient
solution to the thumbnails task.  In this section, we'll introduce
Accelerator features that helps structuring the program and work on a
higher abstraction level with fast execution times.  Mainly, we'll

  - use the Accelerator's build system so that pre-computed
    results are reused and a minimum of processing is required when
    things (source code, parameters, or input data) change;

  - keep track of intermediate results between programs automatically;

  - associate any number of additional information to each image; and

  - reduce the number of intermediate files, and thereby seek time.

The first thing we do is to separate the solution into three different
programs.  We do this to make use of the Accelerator's _dataset_
storage format and to minimise the amount of re-builds when we modify
the code or the input parameters.  Here is a build script reflecting
the partitioning of the code

```python
def main(urd):
    files = ...  # a list of filenames
    job_imp = urd.build('import_images', files=sorted(files))
    job_tmb = urd.build('thumbnailer',   size=(256,256), source=job_imp)
    job_exp = urd.build('export_images', column='thumb', source=job_tmb)
```

The most interesting program is the one in the middle, `thumbnailer`,
that actually computes the thumbnails.  The programs before and after
are just converting to and from the Accelerator's internal format.
The more complicated a processing task, the more this partitioning
makes sense, but we keep to the thumbnails example in this post to
keep things simple.

What about the `sorted()` call?  This is to ensure that we do not
execute any of the programs unless the input data has been modified.
Sorting the input data makes it deterministic, independent of which
order the list of files was generated in.  The `import_images`, and
all jobs depending on its output, will only execute once for a given
input.  It is only when inputs, parameters or source code change that
programs will be executed.  This is a key Accelerator feature.

Before we have a closer look at the `thumbnailer`, let's have a quick
look at the import program



## The Import Program

The import program is much like the first thumbnailing program
presented earlier, but instead of writing output images to files, it
writes to an Accelerator _dataset_.  The dataset is used to store both
the images and some meta information.  In this case the meta
information consists of filenames and sizes.  If we had been
interested in, say, exposure statistics, we could add some or all of
the EXIF-data in addition to filename and size to the dataset.  The
import program is shown in the figure below

<p align="center"><img src="{{ site.url }}/assets/image_tods.svg"> </p>

Each process handles a slice of the list of input files and stores
them in a corresponding dataset slice.  Columns are stored in
independent files so that we can access columns independent of each
other.  Now we move on to the more interesting part.



## The Image Processing part: Thumbnailing

With images and metadata available in a dataset, we can work on a
higher, yet efficient, abstraction level and focus on the actual image
processing flow.  For example, we can send the images to a set of
different image analysis algorithms, generate debug output image sets
for each of them, and merge all computations into a single result.
But in this post we will keep our focus on the thumbnail task.

The figure below illustrates how the `thumbnailer` program works

<p align="center"><img src="{{ site.url }}/assets/image_ds2ds.svg"> </p>

Each one of the parallel processes reads one slice of the `image`
column, creates a thumbnail, and then writes to a new column named
`thumb`.  Note that

- the new column is appended to the existing "source" dataset, and
- there is no need to read any of the `shape` or `filename` data from disk.

After execution, the dataset has four columns.  Three of the columns
were created by the import program and existed before, and one column
is new and created by the thumbnail program.  By appending new columns
to old ones, everything we have computed and know about each image is
being kept together.  Appending new columns to existing datasets is
almost for free, since it is just a matter of linking columns to
datasets.  (And reading relevant columns only is obvious for
performance reasons.)



### Diversion: Computing a Histogram of Image Shapes

It is tempting to show how easy it is to start doing data analysis now
that we have the imported image dataset.  The code below will compute
a histogram of all image shapes

```python
from collections import Counter
datasets = ('source',)
def synthesis():
    return Counter(tuple(x) for x in datasets.source.iterate(None, 'shape'))
```

and we run it by adding this line to the build script

```
    urd.build('shapehist', source=job_imp)
```

Here, `job_imp` is a reference to the import job that was run
previously.  The `shapehist` program will thus always run on the
correct data.  Furthermore, `shapehist` makes use of the available
`shape` column in the dataset (which was generated as a by-product in
the import program).  It does not need to read the images all over
again to compute the shapes, which has an enormous performance
advantage.  (The `tuple(x) for x in ...`-stuff is to make the stored
shapes hashable, otherwise they cannot be used as keys in a Python
Counter.)




## Exporting Images in a Dataset back to Files

Finally, we need a way to generate image files from a dataset.  Such a
program may be visualised like this

<p align="center"><img src="{{ site.url }}/assets/image_fromds.svg"> </p>

The program reads one line at a time (but in all slices) from the
`image` and `filename` columns, and writes the image data into files
with corresponding filename.





# Source Code

The Accelerator's _dataset_ provides efficient store and retrieval of
various kinds of typed data.  Images can be stored simply using the
`pickle` type, that can be used to store any Python pickle-alble data.
(Another way is to store images as raw byte data, but in that case we
need to explicitly create the raw byte sequence from the PIL Image
object.)


## Import


The program starts with a single process executing the `prepare()`
function that sets up a new dataset writer object with three columns.

```python
from os.path import basename
from PIL import Image

options = dict(files=[])

def prepare(job):
    dw = job.datasetwriter()
    dw.add('image', 'pickle')
    dw.add('shape', 'json')
    dw.add('filename', 'unicode')
    return dw
```

The image is stored as a Python pickled object `pickle` and the
filename as `unicode`.  Image shape data is a tuple `(width, height)`,
and is stored as `json` in the dataset.  (We could have used the
`pickle` type for this as well.)  The dataset writer object is
returned by `prepare()` so that it can be passed to the
`analysis`-functions executing next.

The running process is forked into a number of parallel processes
executing the `analysis()` function shown below.  The writer object
returned by `prepare()` is input and referenced by the name
`prepare_res`.  The input `sliceno` holds unique number for the
process, while `slices` holds the total number of parallel processes.

```python
def analysis(prepare_res, sliceno, slices):
    dw = prepare_res
    files = options.files[sliceno::slices]
    for fn in files:
        im = Image.open(fn)
        dw.write(im, im.size, basename(fn))
```

Each `analysis()`-process selects a unique slice of the input filename
list, read the files one a a time, creates an Image object, and writes
the object as well as size and filename to the dataset.





## Thumbnailer

Below is the complete `thumbnailer` program.
```python
from PIL import Image

datasets=('source',)
options=dict(size=(100, 100))


def prepare(job):
    dw = job.datasetwriter(parent=datasets.source)
    dw.add('thumb', 'pickle')
    return dw


def analysis(prepare_res, sliceno):
    dw = prepare_res
    for im in datasets.source.iterate(sliceno, 'image'):
        im.thumbnail(options.size)
        dw.write(im)
```

The dataset writer is fed with a `parent` argument, instructing it to
append columns to an existing dataset instead of creating a new one.
The `analysis()` functions are forked and execute in parallel on all
slices, each iterating over one slice of the input dataset.



## Export

The following program is complete and minimal.  Files are written to
disk in the internal format, i.e. BMP, and the filename extension is
left unchanged.  Files get written into the current job directory.

```python
options=dict(column='thumb')

datasets = ('source',)

def analysis(sliceno, job):
    for im, fn in datasets.source.iterate(sliceno, (options.column, 'filename',)):
    	im.save(fn)
	job.register_file(fn)
```

We get PIL Image objects directly from the iterator, so we can just
use the Image objects `save()` method directly.  Again, we use
`job.register_file()` to make an explicit connection with the created
files and the job.




# Conclusion

This post illustrates how the Accelerator can parallel process binary
image files.  Although the minimalistic design principles behind the
Accelerator has left it without explicit image processing support, it
can be added with just a few lines of code.

The Accelerator brings deterministic processing and re-use of jobs in
order to minimise confusion and processing time, which is a big
advantage when processing for example large sets of image files.
