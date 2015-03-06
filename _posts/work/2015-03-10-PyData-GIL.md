---
layout: post
title: PyData and the GIL
category : work
draft: true
tags : [scipy, Python, Programming]
---
{% include JB/setup %}

*This work is supported by [Continuum Analytics](http://continuum.io)
and the [XDATA Grant](http://www.darpa.mil/our_work/i2o/programs/xdata.aspx)
as part of the [Blaze Project](http://blaze.pydata.org/docs/dev/index.html)*

**tl;dr** Many PyData projects release the GIL.  Multi-core parallelism is
alive and well.

Introduction
------------

The Global Interpreter Lock (GIL) stops multiple threads working on Python
objects in parallel.  This cripples Pure Python shared-memory parallelism.

This sounds like a big deal.  Workstation machines grow more
powerful with more cores every year, providing the same power as a decent
cluster without the hardware hassle.  Moore's law powers on, increasing
transistor density exponentially over time; it does so in the form of
multi-core architectures.

Q: *Given the growth of shared-memory parallelism, should the PyData ecosystem
    be concerned about the GIL?*

A: *No*

Many of the PyData projects don't spend their time in Python code.  They spend
99% of their time in C/Fortran/Cython code.  This code can release the GIL.

The following projects release the GIL:

*  NumPy
*  SciPy
*  Numba in `nopython` mode
*  Scikit Learn
*  (Please feel free to add more in the comments, I'll post them here)

Additionally any project that depends on these gains the same benefit.


Because the PyData stack is fundamentally based on native compiled code
multiple Python threads can crunch data in parallel without worrying about the
GIL.  The GIL does not have to affect us in a significant way.


That's not true, the GIL hurts Python in the following cases
------------------------------------------------------------

### Parsing large text files

We don't have a C/Fortran/Cython solution for text. When given a large pile of
text files we often switch from threads to processes and use the
`multiprocessing` module.  This makes inter-worker communication much more
expensive but this is rarely and issue for this kind of embarassingly parallel
work.

The multiprocessing workflow is fairly simple.  I've written about this in the
[toolz docs](http://toolz.readthedocs.org/en/latest/parallelism.html) and in a
blogpost about
[dask.bag](http://matthewrocklin.com/blog/work/2015/02/17/Towards-OOC-Bag/)

### Pandas

Pandas does not yet release the GIL in computationally intensive code.
It probably could though.  This requires momentum from the community and some
grunt-work by some of the Pandas devs.  I have a small issue
[here](https://github.com/pydata/pandas/issues/8882).