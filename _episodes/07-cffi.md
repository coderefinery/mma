---
layout: episode
title: Using the C Foreign Function Interface for Python
teaching: 20
exercises: 20
questions:
  - Is there a non-intrusive way to couple C/C++/Fortran and Python?
  - Can I test my Fortran code with Python?
objectives:
  - Learn the tools to create a Python interface to almost any C API.
  - Obtain a recipe for testing compiled libraries with Python.
keypoints:
  - Allocate API arrays client-side.
  - Write a thin Python layer around them.
  - Big advantage of CFFI is that the two sides do not need to adapt to each other.
---

## Setting the stage

We will stay with the same pi example but please picture your project instead.

Imagine one of two situations:

- Either you have a Python code and want to create a C/C++/Fortran back-end.
- You have a (possibly legacy) C/C++/Fortran code and wish to create a Python front-end.

---

## Learning goals

FIXME: add batman figure.

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

## Why [Python CFFI](https://cffi.readthedocs.io)?

- General: works with any language that exposes a C API.
- Simple: the interface layer is thin.

---

## Exercise 4: adding a Python interface

First make sure that the C++ and Fortran libraries from the previous session are compiled.

- FIXME SHOW COMPILATION

- FIXME FETCH cffi_helpers.py

The function `get_lib_handle` tells CFFI where to find the header file and the
dynamic library and from this CFFI will create a Python interface.

We have places this function into a separate file so that you can reuse it for
different libraries.

- FIXME FETCH __init__.py

This is the package interface file which exposes 3 functions:

```python
__all__ = [
    'approximate_pi_python',
    'approximate_pi_c',
    'approximate_pi_fortran',
]
```

With these two files we have created a Python interface!

Let us first test it:

FIXME

Why do we need to set `PI_BUILD_DIR` when importing our `pi` package?

FIXME fetch and run timing


After testing the interface, take the time to study the files and discuss the code.

---

## Exercise 5: adding automated testing

### Motivations for testing your C/C++/Fortran code with Python

- Forces you to create a clean interface (good)
- Nice byproduct: you have a Python interface (good)
- Encourages dynamic library (good)
- You can write and prototype tests without recompiling/relinking the library (good)
- Allows you to use the wonderfully lightweight [pytest](http://pytest.org) (no more excuses for the Fortran crowd)

Under construction ...

---

## Exercise 6: adding a setup script

Under construction ...

---

## Memory allocation for arrays that pass the API

![]({{ site.baseurl }}/img/cffi/terminator.jpg)

### Library side

Advantage:

- Caller does not need to know the array sizes *a priori*.

Disadvantage:

- Risk of a memory leak (garbage collection on the Python side).


### Client side (recommended)

Advantages:

- Let the caller decide how memory is allocated.
- Reduced risk of a memory leak.

Disadvantage:

- You need to know array sizes.
- Requires extra layer on the Python side.


### Example

Consider these two functions:

```cpp
double *get_array_leaky(const int len)
{
    double *array = new double[len];

    for (int i = 0; i < len; i++)
    {
        array[i] = (double)i;
    }

    return array;
}

void get_array_safe(const int len, double array[])
{
    for (int i = 0; i < len; i++)
    {
        array[i] = (double)i;
    }
}
```

How can we use the second one?

```python
def get_array_safe(length):
    ffi = FFI()
    lib = ffi.dlopen(...)

    array = ffi.new("double[]", length)
    lib.get_array_safe(length, array)

    return array
```

- Demonstration: [Client-side or library-side memory allocation
  with Python CFFI](https://github.com/bast/cffi-mem-alloc-example).
- You need to hold on to the object for as long as you need the memory on the library-side.


### Questions

- How can you make the API function return a Python list or dict?
- Can you implement generators?
