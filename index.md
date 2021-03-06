---
layout: lesson
permalink: /
---

# Mixed Martial Arts: Interfacing Fortran, C, C++, and Python


## Learning goals

The programming languages Fortran, C, C++, and Python each have their strengths
and weaknesses and their own fan base. This workshop is for people who would
like to be able to combine these languages within one code project:

- When writing a high-level abstraction layer or interface to a "bare metal"
  legacy software written for instance in Fortran or C.
- When writing an efficient back-end to a code mainly written in a high-level
  language such as Python.
- When combining modules written in different programming languages.
- When writing a Python interface to a software in C or C++ or Fortran to
  leverage the wealth of libraries available in Python.
- For testing and prototyping compiled code with Python.

The goal of this lesson is not to become expert in pybind11 or Python CFFI but
to get you on a good track with simple but working examples which can be
adapted to your domain.


## Prerequisites

To appreciate the material it helps to have some previous exposure to Python
and a compiled language (C, C++, or Fortran).


## Required software

This lesson is tested on Linux and Mac.
It should be possible to get it to run on Windows but tweaks are then necessary.

- C++ compiler which is C++11 compliant (e.g. g++)
- Fortran compiler (e.g. gfortran)
- CMake
- Make
- Git
- [pytest](https://pytest.org) and [Python CFFI](https://cffi.readthedocs.io)
  (we recommend to install these using Virtualenv or Anacoda or Miniconda)
- python-dev (Debian-like systems) or python-devel/python3-devel (RedHat-like systems)
