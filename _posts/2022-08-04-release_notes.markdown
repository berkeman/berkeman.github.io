---
layout: post
title:  "Changelog 2022.8.4.dev1"
date:   2022-08-04 00:00:00
categories: changelog
author: Anders Berkeman, Carl Drougge, and Sofia Hörberg
---

## In your methods

- `Dataset.link_to_here()` takes a `rename={'old column name': 'new column name'}` argument.

## Standard methods

- New method `dataset_unbits`.
	- Converts bits32 columns to int64 and bits64 columns to number.
	- Exists because the bits types will be removed in the next release.
- New method `dataset_rename_columns`.
	- Rename columns, like you can do with `Dataset.link_to_here()`.
- `dataset_type` now passes None-values through unchanged.
- `csvimport_zip` can chain `by_dict`.
	- Since dict arguments are now ordered, `inside_filenames` can be used to order your chain.

## Shell

- All boolean arguments to shell commands now have a corresponding off switch.
	- E.g. `--headers` can be disabled with `--no-headers`,
	  `--ordered` can be disabled with `--not-ordered`,
	  `--show-sliceno` can be disabled with `--dont-show-sliceno` and so on.
	  Short form `-X` has corresponding `+X` to disable.
- The `ax curl` command is gone, it is no longer useful.
- New command `ax alias` to list or expand aliases.
- Aliases can contain `${ENV_VAR}`.
- `ax job` can show the source(s) used to build a job.
	- `--source` shows all sources.
	- `--source-file` shows specific files (matching a glob pattern).
- `ax grep` takes a `--lined` switch to alternate colour of output lines.

## And finally

- More tests.
- Various bug fixes / minor improvements of course.

Note: Python 2 is still supported, but will be unsupported soon.
