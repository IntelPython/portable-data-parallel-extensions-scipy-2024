---
title: SciPy 2024 poster content
linkTitle: Details
menu: {main: {weight: 20}}
---

Portable Data-Parallel Extensions with oneAPI
---
by [Nikita Grigorian](https://github.com/ndgrigorian) and [Oleksandr Pavlyk](https://github.com/oleksandr-pavlyk)

This poster is intended to introduce writing portable data-parallel Python extensions using oneAPI.

We present several examples, starting with the basics of initializing a USM (unified shared memory) array, then a KDE (kernel density estimation) with pure DPC++/Sycl, then a KDE Python extension, and finally how to write a portable Python extension which uses oneMKL.
