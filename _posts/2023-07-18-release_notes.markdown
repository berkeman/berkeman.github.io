---
layout: post
title:  "Changelog 2023.7.18.dev1"
date:   2023-07-18 00:00:00
categories: changelog
author: Anders Berkeman, Carl Drougge, and Sofia Hörberg
---

## Highlights

- `job.[json_]save()` takes a `background` argument to let your method do
  other things while saving the file.
- Many usability tweaks in shell commands and board.

## Board

- Support for sub-directories in the result directory.
- Zebra stripe text files and job output.
- Colour support in text files and job output. (By using `accelerator.colour`.)
- Job pages have ⇦ prev, next ⇨ and LATEST ⇉ links.
- Workdir lists can be sorted by clicking the headings.

## csvimport

- `labelsonfirstline` is replaced by a more generic `label_lines`, so multi-line
  labels work (newlines are replaced by spaces).
- If `labels` is set, `label_lines` defaults to 0 (but you can of course set
  it if needed).
- Skipped lines (from labels, `comment`, `skip_lines` and `skip_empty_lines`)
  no longer affect the slicing of included lines. (Bad lines are however sliced
  together with the other included lines, if you set `allow_bad`.)

## In your methods

- job.link_result() works with sub-directories (both source and destination).
- `job.[json_]save()` takes a `background` argument to let your method do
  other things while saving the file.
- `job.[json_]save()` returns an object with some methods and properties:
	- `.load()` (waits until saving is done)
	- `.remove()`
	- `.filename`
	- `.path`

## Shell

- All commands allow `--help` (and sometimes more) without a config file.
- New `ax status` command, showing the currently running method and more
  interestingly the last error (like the board status page already did).
- New `ax gc` command to clean up workdirs. Currently only removes jobs
  that didn't finish.
- `ax job` shows less output by default, adds `--verbose` to get it back.
- The job spec syntax is extended further, and now supports:
	- start with a jobid (e.g. `dev-42`), a method (e.g. `csvimport`) or
	  an urd spec (`:urdlist:[entry]`, e.g. `:import/2023-07-18:`).
	- `~` to go back one job of the same method.
	- `^` to follow .parent.
	- `+` to go forward one job of the same method.
	- `!` to make `~` and `+` only consider current jobs.
	- `<` to go one jobid back (e.g. dev-42 ⇨ dev-41)
	- `>` to go one jobid forward (e.g. dev-42 ⇨ dev-43)
	- `.item` to follow item in argument `jobs` or `datasets`
	  (use `.jobs.item` or `.datasets.item` to disambiguate).

  These can be chained freely, and accept counts where this makes sense. E.g.
  `dataset_type~5.source^` or equivalently
  `dataset_type~~~~~.source.parent`.
  Remember to escape shell special characters (at most `!^<>`).

## Urd

- Newlines (and everything else) are now allowed in captions.
- Urd can run without a config file, by specifying `--listen`.

## And finally

- More tests.
- Various bug fixes / minor improvements of course.

Note: Python 2 is still supported, but will be unsupported soon.
