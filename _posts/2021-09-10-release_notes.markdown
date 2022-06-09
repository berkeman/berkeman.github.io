---
layout: post
title:  "Changelog 2021.9.10.dev1"
date:   2021-09-10 00:00:00
categories: changelog
author: Anders Berkeman, Carl Drougge, and Sofia Hörberg
---

## Highlights

- new `--debuggable` argument to help you debug methods.
- `auto-discover` on method packages, so you can skip `methods.conf`.

## Board

- many small tweaks.

## Configuration file

- you can use `auto-discover` on a method package, making `methods.conf` optional
  in that package.
	- `ax init` now uses this for the package (`dev` by default) you get in the
	new project.

## Dataset

- `dw.add()` can take an existing `DatasetColumn` / tuple (like the `columns`
  dict in the contructor already could)
- `dataset.txt` is no longer produced, as `ax ds` is more convenient anyway.

## Server

- new `--debuggable` argument that runs methods with a controlling terminal so
  that `breakpoint()` and similar will work. This does however make the server
  kill itself if a method fails.

## Shell

- `ds` command updates:
	- new `--location` / `-w` argument that shows the storage location of each
	  column.
	- `--foo_bar` style arguments are now `--foo-bar` (for consistency):
		- `--non-empty-chain`
		- `--suppress-columns`
		- `--suppress-errors`
		- `--suppress-minmax`
- rename `urd` and `board` commands to `urd-server` and `board-server`, for
  consistency and to free up `urd` for:
- new `urd` command to look at your urd db.
- new `script` command that shows information about build scripts.
- job arguments can be specified as `:urdlist:[entry]`.

## And finally

- more tests.
- various bug fixes / minor improvements of course.

Note: Python 2 is still supported, but will likely be unsupported soon.
