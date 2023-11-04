---
layout: post
title:  "Changelog 2023.11.3.dev1"
date:   2023-11-03 00:00:00
categories: changelog
author: Anders Berkeman, Carl Drougge, and Sofia Hörberg
---

## Highlights

- Build scripts now run as jobs.
	- build.main() takes an optional `job` argument (a `CurrentJob`, just
	  like in methods).
	- You can create datasets in build jobs.
	- Jobs now have a `.build_job` property that tells you which build job
	  originally built them.
	- Jobs also  have a `.parent` property that tells you which job
	  originally built them, regardless of what type of job that is.

## In your methods

- You can use `pathlib.Path` (and `pathlib.PurePath`) in options.
- You can pass `Path` objects to accelerator functions that take paths.

## Shell

- `ax grep --unique` which is like `| uniq` but you can do it on only some
  columns (`ax grep --unique=colname [--unique=other_colname ...]`).
- `ax init` takes `--workdir-template` (see example further down).
- `ax init` can take several `--workdir` arguments, which can also be
  in the template format, or `name:TEMPLATE` (or just a name as before).
- `ax run` can run multiple build scripts in one invocation.

## Configuration

- Global /etc/accelerator/config which is read before your user
  .config/accelerator/config.
- Configuration files can include other files, e.g.
  ```
  [include]
      path = /a/file /another/file "relative paths with spaces work too"
  ```
- `ax init` can be configured in the configuration file, e.g.
  ```
  [init]
      workdir-template = /workdirs/[user]/[project]/[name]
      workdir = [name] /workdirs/shared/import
      input = /some/where
  ```

## Build system

- New top level build.sh to simplify building all wheels.
- Make reproducible wheels.
- Update zlib-ng to 2.1.3.

## And finally

- More tests.
- Various bug fixes / minor improvements of course.

Note: Python 2 is still supported, but will be unsupported soon.
