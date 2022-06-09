---
layout: post
title:  "Changelog 2022.5.9.dev1"
date:   2022-05-09 00:00:00
categories: changelog
author: Anders Berkeman, Carl Drougge, and Sofia Hörberg
---

## Highlights

- many new features in `ax grep`.
- new method `dataset_fanout` that creates one dataset per value in a column.
- `dataset_type` can now store unparsable lines in separate dataset.
- you can now configure colours in `$XDG_CONFIG_HOME/accelerator/config`.

## ax grep

- CTRL-T prints progress, much like when building jobs.
	- press it twice quickly for more detailed information.
- explicit `-e <pattern> -d <dataset> -n <column>` args
	- allowing for unambiguous specifications regardless of names.
- support for several patterns.
	- `-e foo -e bar -e ...`
- `--allow-missing-columns` (`-M`)
	- you can specify columns that only exist in some datasets in a chain
	  without problems.
- `--after-context` / `--before-context` / `--context` for both (`-A -B -C`)
	- also print lines after and / or before selected lines.
- `--invert-match` (`-v`)
	- select non-matching lines
- `--list-matching` (`-v`)
	- only print matching datasets / slices.
- `--only-matching` (`-o`)
	- only print the matching part / columns.
- `--tab-length` (`-T`)
	- uses spaces to align fields to this tab length.
- speedups for (partially) ordered cases.
- and some bug fixes too.

## Colours

- you can now configure colours in `$XDG_CONFIG_HOME/accelerator/config`:

	```
	[colour]
		highlight = BOLD RED
		grep/header = UNDERLINE BRIGHTGREEN
		...
	```
- support for 256 colour codes:
	- `#RGB[bg]` (0 - 5) "256 colour" mode.
	- `#GG[bg]` (00 - 1D) grayscale from "256 colour" mode.
	- `XNN[bg]` (00 - FF) directly specifying a "256 colour" index.

## Datasets

- uses less memory in finished `DatasetWriter` (so a method creating many
  datasets after each other uses less memory).
- minor on disk optimisations.

## Standard methods

- `dataset_type`
	- `filter_bad=True` saves bad lines in `/bad` dataset.
- New method `dataset_fanout`
	- creates one dataset per value in a column.

## In your methods

- list items in the `jobs` and `datasets` dicts are now `JobList` or `DatasetList` with
  the same methods as `DatasetChain` (which is now a subclass of `DatasetList`).
- `DatasetLists` now also have a `.range()` function to filter the list to only
  datasets that include some part of a range in a column of your choice.

## Server

- optimisations for memory use and speed with many slices.

## Shell

- `noalias` pseudo-command so you can use the standard version of commands if
  you define an alias using the standard name.

## And finally

- more tests.
- various bug fixes / minor improvements of course.

Note: Python 2 is still supported, but will most likely be unsupported soon.
