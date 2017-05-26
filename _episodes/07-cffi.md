---
layout: episode
title: Testing dynamic libraries with pytest
teaching: 40
exercises: 0
questions:
  - Write me.
objectives:
  - Write me.
keypoints:
  - Write me.
---

## Test your C/C++/Fortran code with Python

- Forces you to create a clean interface (good)
- Nice byproduct: you have a Python interface (good)
- Encourages dynamic library (good)
- You can write and prototype tests without recompiling/relinking the library (good)
- Allows you to use the wonderfully lightweight [pytest](http://pytest.org) (no more excuses for the Fortran crowd)
- Example: https://github.com/bast/context-api-example

---

## Section

Hands-on example for testing a Fortran code with Pytest and deploying the test to Travis CI
