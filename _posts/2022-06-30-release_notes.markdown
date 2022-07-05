---
layout: post
title:  "Changelog 2022.6.30.dev1"
date:   2022-06-30 00:00:00
categories: changelog
author: Anders Berkeman, Carl Drougge, and Sofia Hörberg
---

## Highlights

- Smart tab mode for `ax grep` .
- Dicts in `options` can retain key order.

## ax grep

- Smart tab mode (`-T`/`--tab-length`).
	- Longer separation between fields than the tab interval, so that fields
	  are more likely to stay aligned.
	- On by default for terminal output, use `-T8` for the old behaviour
	  (or use `-t<SEPARATOR>`).
	- Use `-T/` to turn it on for file/pipe output without specifying values.
	- Configure it with `-T<TABLEN>/<FIELDLEN>/<MINLEN>`. Everything aligns to
	  `TABLEN`, fields start at `FIELDLEN` intervals and each value is
	  separated by at least `MINLEN`. Default is 8/16/2.
	- You can also use `NAME=VALUE`, e.g. `--tab-length min=3/field=24`.
- New `--chain-length` and `--stop-ds` options that work like on `ds.chain()`.
	- You can specify them several times if you have several chains (i.e. have
	  specified several datasets to chain from).

## Server

- Rename `--debug` option to `--keep-temp-files`, to reflect the only remaining
  functionality of that option.

## In your methods

- `DotDicts` (which all dicts in `options` are) are now `OrderedDicts` and if
  you pass an `OrderedDict` in the call to the method order is preserved.
  (If you pass a normal dict the keys will be sorted.)
- Unset `job`/`dataset` values now provide some of the methods and properties
  a set value would have with empty results. E.g.:
	- `jobs.foo.datasets() == DatasetList([])` if `jobs.foo` is unset.
	- `jobs.foo.files() == set()` if `jobs.foo` is unset.
	- `jobs.foo.load() == None` if `jobs.foo` is unset.
	- `datasets.bar.lines[n] == 0` if `datasets.bar` is unset.
	- `datasets.bar.columns == {}` if `datasets.bar` is unset.
	- Most other properties are `None` or `''`.
- `job.datasets` is a `DatasetList`, so you can use it like a `DatasetChain`.
- New `job.finish_early(result=None)` function to end a job early (without
  running later steps).
- `DatasetWriter` now supports `default=` on all column types.

## And finally

- More tests.
- Various bug fixes / minor improvements of course.

Note: Python 2 is still supported, but will most likely be unsupported soon.
