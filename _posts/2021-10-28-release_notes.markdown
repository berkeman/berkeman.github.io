---
layout: post
title:  "Changelog 2021.10.28.dev1"
date:   2021-10-28 00:00:00
categories: changelog
author: Anders Berkeman, Carl Drougge, and Sofia Hörberg
---

## Highlights

- new `intro` command to help you get started.
- examples:
	- use `ax init --examples` to get examples of simple methods in your
	  new project.
- shell commands can output csv or json from a dataset.

## ax grep

- new `--format=` / `-f ` argument that accepts
	- `text` (the old default)
	- `csv` (the new default, similar to text but with quoting when needed)
	- `json`
- coloured matches are now the default if output is a tty, `--colour` argument
  now works similarly to gnu grep (`auto` / `never` / `always`).

## Build

- new `force_build` flag, available as an argument on `build()` calls and also
  as "`ax run --flags force_build`" (to force build everything in that script).

## Datasets

- better error messages for many problems in writers.
- `Dataset.min(colname)` and `.max(colname)` to match the API on
  `DatasetChain` (including giving `None` for non-existant columns).

## Shell

- new `intro` command to help you get started.
- `init` can include `--examples` in the new project.
- `script` command takes `--path` / `-p` argument.
- `run` command changes `--fullpath` / `-F` to `--full-path` / `-p` for
  consistency.
- `workdir` command takes `--full-path` / `-p` argument.
- all commands should now only use colour when output is a tty, or as specified
  with the env vars `NO_COLOR`, `CLICOLOR` and `CLICOLOR_FORCE`.

## And finally

- more tests.
- various bug fixes / minor improvements of course.

Note: Python 2 is still supported, but will likely be unsupported soon.
