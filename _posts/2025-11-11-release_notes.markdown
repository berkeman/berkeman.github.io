---
layout: post
title:  "Changelog 2025.11.11"
date:   2025-11-11 00:00:00
categories: changelog
author: Anders Berkeman, Carl Drougge, and Sofia Hörberg
---

This covers important changes since the 2024.6.11 release, as nothing was
written about the releases between then and now. They were mostly bug fix
releases.

## Highlights
- `analysis_res.merge_auto()` no longer allows overwriting dict items by default.

## Board
- The urd item and build script views shows files that were `link_result()`ed.

## In your build scripts
- Urd now allows any number of timestamp pieces in any order. Previously it
  was exactly one (optional) timestamp followed by one (optional) integer.
  Now you can use horrors like `7+2025-11-03+2024-05-14+1+2+3` if you need to.
- `job.link_result()` now takes optional `header` and `description`
  arguments that will decorate the result in board's urd item and
  build script views.

## In your methods
- `analysis_res.merge_auto()` is not new, and does mostly the same thing it has
  always done, but now it will raise an `Exception` if there are duplicate keys
  in the innermost level of dicts. (It is still allowed in sets and Counters.)
  There is a new `allow_override` parameter to control this behaviour.

## And finally

- More tests.
- Various bug fixes / minor improvements of course.

Note: This is the last release to support Python 2.7 and 3.5.
