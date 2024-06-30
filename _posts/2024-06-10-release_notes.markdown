---
layout: post
title:  "Changelog 2024.6.10"
date:   2024-06-10 00:00:00
categories: changelog
author: Anders Berkeman, Carl Drougge, and Sofia Hörberg
---

## Datasets

- datetime and time columns will return `tzinfo=UTC` if written with an
  UTC-like `tzinfo`, and will refuse to write if a non-UTC `tzinfo` is given.
  (Previously `tzinfo` was discarded.)
- Calling `dw.finish()` several times is now permitted, which can sometimes be
  convenient.
- Hashing of complex values of the form `-0+anything` and all float types with
  `NaN` in them should now hash the same on all systems. This unfortunately
  means that they may not hash the same as they previously did on your system.

## dataset_type

- If you set a timezone datetimes (but not times) will set `tzinfo=UTC`.
  (This is true regardless of which timezone you specified. Like before
  they are converted to UTC, the difference is they are now returned as
  timezone aware objects.) This also changes specifying `timezone='UTC'` into
  something sensible, and this is now special cased to not have any caveats.

## In your build scripts

- New `urd.joblist_all` which is like `urd.joblist` but never cleared within a
  build script.
- Everything built (i.e. `urd.joblist_all`) is saved in the urdlist
  `__auto__/$STARTTIME` unless there is an error (or the list is empty).
- The build job that creates an urdlist is recorded. (If a new build job
  produces the same list this information is not currently recorded
  anywhere.)

## In your methods

- If a job does not register any files (including automatically by using e.g.
  `job.open()`) all files that are not in a subdirectory or begin with `.` will
  be registered automatically.
- New `job.register_files(pattern='**/*')` to bulk-register files. (On python 2
  it's not recursive as `glob.glob()` is not.)
- New `job.link_to_here(filename='result.pickle')` to link files from subjobs
  (like you already could with datasets).

## Board

- The job output display supports sixels. (So you can e.g. call
  `GNUTERM=sixelgd gnuplot ...` and see the graph in board.)

## And finally

- More tests.
- Various bug fixes / minor improvements of course.

Note: Python 2 and 3.5 are still supported, but will be unsupported soon.
