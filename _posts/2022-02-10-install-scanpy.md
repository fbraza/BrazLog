---
title: Installation of Scanpy on a Mac M1
hide: false
toc: false
comments: true
layout: post
description:
categories: [Python, DevOps, single-cell]
---

Here a short article to provide some reminders about how to install Scanpy on your Mac M1. For managing my environments I use `poetry`, `conda` or `pipenv`. Unfortunately I did not find manage to install Scanpy via `pip` depedent package managers (both `pipenv` or `poetry` do not work). The only alternative that worked is with`conda`. To be complete there are some dependencies and configurations that need to be set first.

## HDF5

You first need to install `hfd5`. It is a high performance data software library and file format to manage, process, and store your heterogeneous data. HDF5 is built for fast I/O processing and storage. Scanpy can read and store AnnData object as `h5ad` files taht `hdf5` files with some additional structure specifying how to store AnnData objects. To install it you will need homebrew on your Mac M1 machine.

```bash
brew install hdf5
export HDF5_DIR=/opt/homebrew/Cellar/hdf5/1.12.1 # use the version you have
```

## LLVMLITE

Next install `llvmlite` with homebrew. `llvmlite` provides a Python binding to LLVM for use in Numba that translates a subset of Python and NumPy code into fast machine code. It is extensively used in Scanpy given its dependency to numpy and scikit-learn.

```bash
brew install llvm@11
```

Make sure `/opt/homebrew/opt/llvm@11/bin` is in your path. For that edit the `/etc/paths` to add this path. Next install `llvmlite` in your python environment.

## Conda

I recommend installing Miniconda. Miniconda is essentially an installer for an empty conda environment, containing only Conda, its dependencies, and Python.

```bash
conda config --add channels defaults
conda config --add channels bioconda
conda config --add channels conda-forge
conda config --set offline false
```

Once done, you can install `scanpy` in your machine.

```bash
conda install scanpy
```

## Conclusion

It was a short one, but it will be, for sure, a good reminder for my future `scanpy` projects when working in a Mac M1. I hope that in the future these issues will be solved. In the mean time I still use my Linux computer when I need to deal with Scanpy. There everything works out of the box with classical `pip install`.
