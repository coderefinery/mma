---
layout: episode
title: Overview over different approaches
teaching: 5
exercises: 0
questions:
  - Which tools are out there?
objectives:
  - Get a birds-eye overview of existing technologies.
keypoints:
  - Many tools exist with different scope, expressiveness, and complexity.
---

## Overview

There exist several tools which make it possible to couple Python and compiled libraries:

|          | Pros                      | Cons                             |
|----------|---------------------------|----------------------------------|
| Cython   | powerful                  | complexity                       |
|          |                           | requires writing interface files |
|          |                           | complicates build process        |
|----------|---------------------------|----------------------------------|
| pybind11 | header-only               | requires C++11                   |
|          | lightweight               |                                  |
|          | great for C++11           |                                  |
|          | C++ can call Python       |                                  |
|----------|---------------------------|----------------------------------|
| SWIG     | supports many languages   | requires writing interface files |
|          |                           | complicates build process        |
|----------|---------------------------|----------------------------------|
| Boost    | powerful                  | huge                             |
|----------|---------------------------|----------------------------------|
| F2PY     | direct link to Fortran    | intrusive to source code         |
|          | no need to go via C       | restricts some features          |
|----------|---------------------------|----------------------------------|
| CFFI     | general                   | for C++, pybind11 is more direct |
|          | simple                    |                                  |
|          | non-intrusive             |                                  |
|          | requires C interface      |                                  |
|          | generates Python bindings |                                  |
|          | any language (via C)      |                                  |

In this tutorial we will take a closer look at two of these tools:

- pybind11: it is a modern, simple, and lightweight way to interface Python and C++.

- CFFI: because it is modern, simple, non-intrusive (does not require modifying
  neither the Python nor the compiled code), and very general since almost any
  language can talk to a C interface.
