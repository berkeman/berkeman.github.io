---
layout: post
title:  "PyData Global Talk: Computations as Assets - a New Approach to Reproducibility and Transparency"
date:   2021-10-28 00:00:00
categories: talk
author: Anders Berkeman, Carl Drougge, and Sofia Hörberg
---


## PyData Global 2021

**NOTE: A recording of the talk will be available from PyData early 2022!**

The ExAx open source project from eBay provides reproducibility,
transparency, and fast parallel processing in Python.  This talk will
show how reproducibility by design actually leads to a simpler and
faster development process.  To make our examples more interesting, we
will use relatively large datasets such as those from NYC Taxi and
Backblaze.  Description


- Have you ever been struggling to keep order in an increasing set of
temporary data files, holding intermediate results corresponding to
different steps and versions of the processing?

- Have you ever been in a situation where you have tried playing with
parameters and small code changes to see how it affects the program's
performance, and then cannot remember exactly which setting that
performed the best?

- Have you ever re-run your complete processing pipeline just to "make
sure" that the graph you are looking at actually comes from the data
and source code you thought it did?

The ExAx project is designed to avoid problems like these. It treats
computations as assets, tagging computed results with links to input
data and source code, and stores them permanently on disk in a way
that can be easily looked up and retrieved later. In addition, ExAx
provides a simple way to parallel process large datasets in Python on
a single computer.

ExAx is open source from eBay. It runs on anything from laptops to
rack servers, it can be used in a production environment with multiple
users, and it can easily handle datasets with tens of billions of
rows.

Link to [PyData Global](https://pydata.org/global2021/schedule/presentation/44/computations-as-assets-a-new-approach-to-reproducibility-and-transparency/)  
Link to the corresponding [source code at github](https://github.com/exaxorg/backblaze_animation)
