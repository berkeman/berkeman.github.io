---
layout: post
title:  "Being Reproducible:<br>  From Input Data to Result and Back"
date:   2020-07-01 00:00:00
categories: example
author: Anders Berkeman, Carl Drougge, and Sofia Hörberg
---

# TL;DR

The Accelerator eliminates the need for manual naming and book keeping
of intermediate results and dependencies.  Instead, it stores all job
input, code, and output to disk, and provides a simple system to look
up any pre-computed job.  This storage and retreival of job
information may be used to build complex job dependencies that are
transparent, straightforward to validate, and reproducible.



# Introduction

A key feature of the Accelerator from eBay is its *build system* and
the way it stores and keeps track of all programs executed and their
corresponding output.  A design goal of the Accelerator is to provide
**transparency** and **reproducibility**, to any project.  While these
are main features on their own, the design also reduces the risk of
making manual errors, while typically speeding up execution times,
both in the development and deployment phases.

The improvement in speed is because the Accelerator ensures that the
_same program can not be executed twice on the same data_.  If a
result has been computed before, it will be re-used instead of
re-computed.  (Another, unrelated, reason for the Accelerator to being
fast is that it provides a parallel streaming data interface that it
is embarrassingly simple to write parallel programs around, in plain
Python.)

The figure below is a high level illustration of what the
Accelerator's processing flow looks like

<p align="center"><img src="{{ site.url }}/assets/input_results_splash.svg"> </p>

Data processing is "from left to right", starting with input data end
ending with output results.  In between, any intermediate or temporary
storage is written (and read back) to a dedicated space called the
*work directory*.

All intermediate data associated with what is stored in the result
directory can be examined in detail using the Accelerator's *board*
web server.  The Accelerator has always provided transparency, and the
board server makes it easy to observe jobs, files, source code,
datasets, dependencies, and more using a web browser.  The board web
server is the topic of an upcoming post.

<p align="center"><img src="{{ site.url }}/assets/board.png"> </p>

This post explains how the Accelerator's build system is designed to
map input data, source code, and results together.  But first we'll
take a tour of common problems and solutions.



# Result = Input + Program + Execution time

Let us start with the basics.  Typically, a data science task is about
creating some output from a data set, like this

<p align="center"><img src="{{ site.url }}/assets/input_output.svg"> </p>

The output could be some insight in form of a graph or number, some
kind of model of the data, a subset of the data, or something else.
It does not matter what it is, the point here is that the result is a
function of the input data.

Conceptually, the output is created by a program acting on the input,
something like in the figure below

<p align="center"><img src="{{ site.url }}/assets/input_prog_output.svg"> </p>

From the image, it is obvious that the result does not only depend on
the input data, but also on the *actual program* used to generate the
result.  If the program takes input options, the result may
depend on these as well.

We also note that it takes some _execution time_ to generate the
result from the input data using the program.  If the program runs
fast and the dataset is small, this is perhaps not a big issue.  (A
classic solution to keep track of the process from input to output is
to use a
[_Makefile_](https://www.gnu.org/software/make/manual/make.html).
When the source code is updated we can update the result
correspondingly by running `make`.)  But if the program runs for a
longer time, development cycles will be longer and results will be
more valuable since it takes time to generate (and perhaps
re-generate) them.  *Therefore, it makes sense to store all computed
results and associate them with the source code and input data that
was used to generate them.*



## Divide and Conquer

Development and debugging of a program becomes increasingly difficult
with larger datasets and longer execution times.  To overcome the
issue of longer execution times, it is common to use an iterative
approach.  The program can typically be split into different parts,
where input to one part is the output of another, like this.

<p align="center"><img src="{{ site.url }}/assets/input_prog_output2.svg"> </p>

The main benefit here is that once we've developed and debugged
`program1`, we can store its output on disk and concentrate on the
development of `program2` _without_ running `program1` again.  Assuming
that writing and reading the intermediate storage is fast, this may
save a considerable amount of execution time.

Another benefit might be that the output data from `program1` may be
used for several applications, so we can fork different sub projects
from the output of `program1` instead of starting over with the input
data file again.  If `program1` does some data preprocessing or
cleanup, for example, this makes total sense.  In fact, we probably
want to _ensure_ that the same preprocessing is applied to all our
sub projects.  "Don't Repeat Yourself".



## The Problem of Versions

While program partitioning, as shown in the previous section, is a
common approach that seems simple enough, it is actually a potential
road to trouble.  Why is that?  Because it usually does not take long
before we have a situation like in the figure below (or even more
complex)

<p align="center"><img src="{{ site.url }}/assets/input_prog_output3.svg"> </p>

Here, we've made two changes.

 - First, `program1` was modified.  This modification caused the output
to change as well.  The updated output had to be stored on disk so
that `program2` could use the updated instead of the old version.
Running `program2` on the updated result caused the output result to change.

- Second, `program2` was modified.  This caused the result to change again.

In total, we now have three different results.  There are several
potential problems here, for example (tick the ones you recognize):

- We have to manually map the different results to the different
  versions of the programs that we have run.  Potentially, the input
  data has been modified too, so we need to keep track of the data
  version as well.  Manually.

- Maybe we have overwritten old results with new ones during
  re-execution, and in that case there is no previous output left for
  us to compare our latest results to.  Maybe we did a copy/paste of
  some important numbers to a text file or spread sheet.  But which
  version of the code did it match again?

- Are we sure the result is based on the current version of the data
  and code?  If not - run everything again just to be sure (*sigh*)!

- What if we decide that the first version is the one we want to keep?
  If we've overwritten our intermediate data, we have to re-run the
  first version *again*.  Time and energy consuming.

  Could we have kept all outputs and results?  - Yes we could - disk
  space is very cheap these days.

It seems we have to _remember_ or _manually keep track of_ which
result that belongs to which version of the code.  **This does not
scale well**, not with complexity, and not over time, since we
probably forget what is what after a while.

Apparently the approach is not *transparent*, not necessarily
*reproducible*, and since it relies on human book keeping it is
definitely *easy to make mistakes*.

As we'll see next, the Accelerator's build system provides *simple*
solutions to these problems.



# The Accelerator Approach

The approach taken by the Accelerator is simple and efficient.

The Accelerator has the concept of _jobs_.  Everything related to a
single program run, all inputs and outputs as well as the program
itself, is collected and stored in *one place* for reference and
transparency.  This is called a _job_.  The concept of jobs ensure
that *any intermediate files in a project are tied to the job that
created them.*

Jobs can only be created, not modified or erased, by the Accelerator.
Furthermore, they can only execute once.  When a program completes
execution, the Accelerator returns a Python object that contains
references to all data related to the job.  The returned Python object
can be input to new jobs, creating _dependencies_, so that they can
make use of the job's data.   Dependencies are clearly stated in
the jobs' metadata, and are thus fully transparent.
Below is a simple example.

```python
def main(urd):
    job1 = urd.build('import_datafile', filename='data.txt')
    job2 = urd.build('process_data', source=job1)
```

Some files (such as diagrams, toplists, movies, et cetera) in some
jobs are more important and considered to be project _results_.  These
files can be made accessible from the `result directory` using soft
file system links.  Here is a simple code snippet for demonstration

```python
def main(urd):
    job = urd.build('myprogram')
    job.link_result('theoutput.txt')
```

This so called _build scrip_ first executes `myprogram`, and then
links the generated output file `theoutput.txt` to the *result
directory*.  No files are stored in the result directory, it will only
contain *soft links* to files located in jobs.  Links provide
transparent pointers to the jobs that created the resulting files,
whereas a plain file copy does not.

The figure below is a high level view of the Accelerators data flow

<p align="center"><img src="{{ site.url }}/assets/input_results.svg"> </p>

The _input directory_ keeps all input data files.  All jobs are stored
in the _work directory_ as plain files - a filesystem is a database
too, and one that is easy to inspect!  And finally, major results are
linked to the _result directory_.



## Transparency

By looking at a link in the _result directory_, one can directly see
which file inside a job directory that it points to.  Inside this job
directory is the source code that was used to generate the
file, along with references to input data and other jobs that were
used to process the data.  *Following all links backwards will unwind
the complete processing graph, independent of how complex it is, all
the way back to the input data*.  For each job, source code and
options can be extracted too.  **Using the Accelerator, there
is _always_ a 100% transparency from output to input.**

All jobs leading to a link in the result directory, as well as their
properties, are easily browsed using the Accelerator's **board** web
server.



## So What Exactly is Stored in the Job Directory?

In practice, when a new program is run, the Accelerator creates a
directory (called a _job directory_), which is populated with files
covering all information that relates to the program run.  The first
things that are written in the directory are files contain meta
information about the job.  Thereafter, any files written by the
running program as well as the program's return value and print
strings is stored in the directory.  Thus, the job directory contains
among other things

 - the program's source code,
 - all input parameters,
 - everything written to stdout and stderr,
 - all output files generated by the running program,
 - the return value of the program,
 - execution time and profiling information, and
 - a hash digest uniquely identifying the job.

Thus, a job directory contains everything needed to run a specific
program with a particular set of input parameters and data references.
And also everything output during the program's execution, including
temp files, print-outputs, stderr-messages and return value.



## Re-use, don't re-compute!

Note that _jobs can only be built once_.  For every build-call, the
Accelerator checks if there is a corresponding job already built.  If
there is, the Accelerator will immediately return a job object
containing references to the job.  This job object looks exactly the
same as the one returned the first time, when the job was actually
built.

The Accelerator finds pre-built jobs in an instant, because each job
stores a hash digest of the job's source code.  The hash digest
together with input parameters is a unique identifier of a job.

If a build script is fetching pre-built jobs instead of building new
ones, it actually validates all processing from input to output.  A
re-build, on the other hand, is a sign saying that something has been
modified since last run.  Storing and looking up jobs saves time while
ensuring transparency and reproducibility.  So **re-use, don't
re-compute**.



# About the Accelerator

The Accelerator from eBay is designed for fast processing of large
datasets on a single machine.  Programs are simply written in Python,
and thanks to its fast parallel processing and clever reuse of
pre-computed results, execution times are typically in the range of a
few seconds per task.  Therefore, ideas could be tried out with very
little overhead, making the Accelerator a perfect tool for fast
exploration of large datasets.

