---
layout: post
title:  "Changelog 2021.4.1.dev1"
date:   2021-04-01 00:00:00
categories: changelog
author: Anders Berkeman, Carl Drougge, and Sofia Hörberg
---

## Init
- `ax init` now creates a git repository by default.  This can be overridden using a switch
- `ax init` can configure tcp port usage (`--tcp`) (unix sockets by default)

## Configuration file
- updated Board and Urd server options

## Standard methods
- `csvexport`
  - lazy quoting
  - selectable None-string (per column if you want)
- `csvimport_zip`
  - improved status messaging when pressing CTRL-T

## Datasets
- bugfix in dataset `min`/`max` and `NaN`-values
- copy-mode allows data to be read and copied in its internal format.
  This significantly speeds up methods like `dataset_sort`, `dataset_hashpart`, and similar.
- allow empty column name.
  Pandas likes to write these.

## Server
- use `time.monotonic` almost everywhere for more stable exec time
  reporting
- improved error handling when terminating unexpectedly
  such as when running out of memory
- concurrency setting.
  It is now possible to set the max allowed concurrency in
  `build()`-calls and from the command line (for all or specified
  methods).  This helps running for example memory intensive
  methods on a machine with limited memory.
- spaces (and unicode on Python3) works "everywhere".
  workdirs, methods, and package names could have spaces and unicode
  characters if the underlying file system supports it.
- various optimisations and fixes

## Board
- `JobWithFile` presented with jobs and files as clickable links
- a new hide button that temporarily remove a specific result
- various tweaks

## Tests
- lots of new tests added

Note: Python 2 is still supported, but will most likely be unsupported
after the release of Python 3.10.
