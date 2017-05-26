---
layout: episode
title: Roadmap for migrating and modularizing legacy code
teaching: 10
exercises: 0
questions:
  - Big untested legacy monolith code in front of you - what now?
objectives:
  - Offer a step-by-step recipe for migrating and modularizing legacy code.
keypoints:
  - Introduce testing early as a safeguard.
  - Use code coverage analysis tools.
  - Cut the code at important interfaces.
  - Build and test modules separately to identify hidden dependencies.
---

## Suggestion for a roadmap


### 1) Add end-to-end test

This means feeding the code with input, waiting for the result, and compare the result against a reference.

- This should be scripted.
- Do not touch the code before you have a test as safeguard.


### 2) If possible, add unit tests

- It is unrealistic to add unit tests for every single function.
- Test central functions and functions which are important entry points between modules/sections.


### 3) Test coverage, iterate until test coverage is sufficient

- Use tools like [gcov](https://gcc.gnu.org/onlinedocs/gcc/Gcov.html).
- Use platforms like [Coveralls](https://coveralls.io).
- Identify untested code and dead wood.


### 4) Define interfaces

Identify important entry points between modules/sections. This is an iterative process.


### 5) Isolate modules behind interfaces

Make interface functions publicly accessible and make all other functions publicly inaccessible.

This is a painful step since you will identify many dependencies.


### 6) Refine interfaces

Iterate steps 4 to 6.


### 7) Localize global data

Bad:

```python
my_parameter = ...

def function1(...):
    # uses my_parameter
    return ...

def function2(...):
    # uses my_parameter
    return ...
```

Better:

```python
def get_my_parameter():
    ...
    return my_parameter

def function1(my_parameter, ...):
    # uses my_parameter
    return ...

def function2(my_parameter, ...):
    # uses my_parameter
    return ...
```

### 8) Build modules separately into libraries

This will expose dependencies.


### 9) Test modules separately

- Speeds up the edit-test-commit loop and helps exposing dependencies and sharpening the API.
- The minimum is to test the API.


### 10) Minimize dependencies

The less dependencies the better.


### 11) Maximize cohesion

Bad:

```python
def does_a_or_b(..., option_b=False):
    return ...
```

Better:

```python
def does_a(...):
    return ...

def does_b(...):
    return ...
```


### 12) Document and version interfaces

- It is often unrealistic to document all functions.
- Interfaces need to be [versioned](http://semver.org) and documented.
- Interfaces hopefully do not change often, inner functions do.


### 13) Outsource modules into own repositories

- Large enough independent units should track own development history.
- This allows to incorporate them in other projects without duplicating code.


### 14) Include external repositories with the help of CMake and possibly git submodules/subtrees

- [CMake ExternalProject](https://cmake.org/cmake/help/latest/module/ExternalProject.html)
- [Git submodules](https://git-scm.com/book/5_submodules.html)
- [Git subtrees](https://medium.com/@porteneuve/mastering-git-subtrees-943d29a798ec)
