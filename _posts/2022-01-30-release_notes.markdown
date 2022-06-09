---
layout: post
title:  "Changelog 2022.1.30.dev1"
date:   2022-01-30 00:00:00
categories: changelog
author: Anders Berkeman, Carl Drougge, and Sofia Hörberg
---

This changelog is really for the 2022.1.28.dev1 release, but don't use that one.

## ax grep

- rename `-o` (short form of `--ordered`) to `-O`.
- remove `-C` (short form of `--colour`).

this is to prepare for using those forms for the same thing gnu grep uses them
for.

## Datasets

- improved on-disk format:
	- dataset name can now be empty.
	- small numbers in number columns are more efficient.
	- each column has a specified compression method, though only gzip is
	  supported for now.

## In your methods

- `DatasetWriter` accepts an `allow_missing_slices=True` argument, which makes
  missing slices (when using `.set_slice()`) allowed.

## Shell

- the job/ds argument parser now works with `:urdlist:entry/dsname`.

## And finally

- more tests.
- various bug fixes / minor improvements of course.

Note: Python 2 is still supported, but will likely be unsupported soon.
