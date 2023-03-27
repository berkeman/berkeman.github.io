---
layout: post
title:  "Changelog 2022.3.10.dev1"
date:   2023-03-10 00:00:00
categories: changelog
author: Anders Berkeman, Carl Drougge, and Sofia Hörberg
---

## Highlights
- `dataset_type` and `dataset_hashpart` now produce one dataset for each
  source dataset.
- New datetime parser, no longer uses strptime from libc.
  This adds several new features and will probably not break existing uses.
	- No longer allows spaces around each value, you can put spaces in your
	  pattern if needed. Fixed length numbers are still allowed to use spaces
	  as leading zeros though. This is the most breaking change.
	- Wildcards for various character classes:
		- `% ` whitespace (exactly one, plain space is any number)
		- `%.` any character except whitespace
		- `%*` any character including whitespace
		- `%#` any digit
		- `%@` any non-digit character, excluding whitespace
		- `%^` any non-digit character, including whitespace
		- These also accept a `COUNT` or `LOW,HIGH` range,
		  e.g. `%2,5.` matches 2 to 5 non-whitespace characters
	- `%f` also accepts a `COUNT`, `%6f` is the same as the default
	  (microseconds), `%3f` is milliseconds etc.
	- Optional elements:
		- `%?` makes the next element (or `COUNT` or `LOW,HIGH` elements)
		  optional. If less than LOW tokens match, no part of the string is
		  consumed, no updates occur to the parsed value and matching
		  continues after HIGH elements in the format string.
		- `%:` is `ELSE` (like `:` in `?:` in C), only executes the following
		  group if the previous `%?` did not match.
		- `%-` ignores the next element value. It must still parse as the
		  specified type, but the parsed value is discarded.
		- `%/` makes the rest of the pattern optional. Matching continues
		  until it fails. Unconsumed input is still an error unless you have
		  chosen to ignore trailing text using an `i`-suffixed type.
	- New format code `%e` for "Excel dates". Three possible variants:
		- `%0e` LibreOffice dates, epoch is 1899-12-30
		- `%1e` Lotus 1-2-3 dates, epoch is 1899-12-31 and 1900 is a leap year.
		  (This is the default, since that is what Excel uses.)
		- `%2e` Excel Mac dates, epoch is 1904-01-01.

## In your methods
- `job.load()` and `job.load_json()` now take a `default=` argument specifying
  an object you get back (instead of an exception) if the file does not exist.
- `job.save()` and friends never complain about missing temp= arguments, these
  silently default to False everywhere now.
- Jobs now save what `input_directory` they ran with.
	- Available as `job.input_directory` and also in `ax job`.
	- Changing `input_directory` still does not cause any jobs to be rebuilt.
- Report the line reached when an exception occurs while iterating a dataset.
- Datasets no longer support the `bits32` and `bits64` types.
- Datasets have a `.chain_within_job()` method. (Like `.chain()` but stopping
  as soon as the previous dataset is in a different job.)
- The new datetime parser is exposed as `accelerator.dsutil.strptime()` and
  `accelerator.dsutil.strptime_i()`.

## Standard methods
- `dataset_type` and `dataset_hashpart` now produce one dataset for each
  source dataset. This removes a lot of corner cases with datasets that differ
  across the chain.
- The `as_chain` argument of `dataset_type` and `dataset_hashpart` is now
  `chain_slices`, to better match what it means.
- More changes in `dataset_type`:
	- Uses the new datetime parser.
	- Remove the `floatint` types, they were too specialised for general usefulness.
	- All columns can be renamed, not just typed ones.
	- Typing errors now show the source ds, column name and sliceno  instead
	  of the much less useful filename.
- New method `dataset_concat` which can be used after `dataset_type` or
  `dataset_hashpart` to get a result more similar to the previous behaviour
  (but without support for varying columns).
- Remove the `dataset_unbits` method again. (Only in the `2022.8.4.dev1` release.)

## Shell
- `ax ds --location` (`-w`) shows where a column originated. Both parent
  dataset and if applicable where it was typed from.
- `ax job` can show files produced by a job with `--file` (`-f`).
- Three new options for `ax grep`:
	- `--fixed-strings` (`-F`), like on standard grep.
	- `--max-count` (`-m`), like on standard grep.
	- `--numeric` (`-N`) to match (ranges of) numeric values instead of text.

## And finally

- More tests.
- Various bug fixes / minor improvements of course.

Note: Python 2 is still supported, but will be unsupported soon.
