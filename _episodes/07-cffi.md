---
layout: episode
title: Using the C Foreign Function Interface for Python
teaching: 20
exercises: 20
questions:
  - Write me.
objectives:
  - Write me.
keypoints:
  - Write me.
---

## Setting the stage

We will stay with the same pi example but please picture your project instead.

Imagine one of two situations:

- Either you have a Python code and want to create a C/Fortran back-end.
- You have a (possibly legacy) C/C++/Fortran code and wish to create a Python front-end.

---

## Learning goals

### Specific goals

- Approximate pi using the Monte Carlo method
- Calling Fortran/C(++) libraries from Python using [Python CFFI](https://cffi.readthedocs.io)
- Automatically testing Fortran/C(++) libraries on Linux and Mac OS X using
  [pytest](https://docs.pytest.org) and [Travis CI](https://travis-ci.org)
- Hiding CMake infrastructure behind a simple `pip install`


### Big-picture goals

- Automatically test dynamic Fortran/C(++) libraries
- Write tests without recompiling the code
- Speed up your Python code
- Provide a Python API to your compiled library and leverage Python tools

---

## Motivations for testing your C/C++/Fortran code with Python

- Forces you to create a clean interface (good)
- Nice byproduct: you have a Python interface (good)
- Encourages dynamic library (good)
- You can write and prototype tests without recompiling/relinking the library (good)
- Allows you to use the wonderfully lightweight [pytest](http://pytest.org) (no more excuses for the Fortran crowd)
