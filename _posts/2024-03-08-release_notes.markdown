---
layout: post
title:  "Changelog 2024.3.8.dev1"
date:   2024-03-08 00:00:00
categories: changelog
author: Anders Berkeman, Carl Drougge, and Sofia Hörberg
---

## Highlights

- Board shows graphs of job / dataset / urd list relationships.
- New `ax hist` command that shows a histogram (or toplist of values) for
  column(s) in datasets.
- New `ax sherlock` command to find out which job a file fetched from board
  originated in.

## Board

- Show graphs of job / dataset / urd list relationships.
- Add metadata to files fetched from jobs (for supported formats), so you
  can later find out which job it came from.

## csvimport

- New option `label_encoding`.

## dataset_type

- All numeric types now reject empty strings (no longer parsing that as 0).
- All types can now be `type+None`, which converts the empty string to `None`.

## Shell

- New `ax hist` command that shows a histogram (or toplist of values) for
  column(s) in datasets.
- New `ax sherlock` command to find out which job a file fetched from board
  originated in.
- `ax grep --roundrobin` which is like `ds.iterate('roundrobin')`, so you can
  e.g. get the original order from a csvimport.
- `ax grep` takes `--dont-grep` (`+g`) and `--not-column` (`+n`) to use
  all columns except those specified.
- `ax init` now ignores `venv` and `.venv` directories in the project directory.

## And finally

- More tests.
- Various bug fixes / minor improvements of course.

Note: Python 2 is still supported, but will be unsupported soon.
