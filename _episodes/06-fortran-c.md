---
layout: episode
title: Fortran, C and C++ talking to each other
teaching: 10
exercises: 10
questions:
  - How can we couple Fortran with C/C++?
  - What is name mangling?
  - How can we avoid worrying about it?
  - How can we safely pass data types across languages?
objectives:
  - Learn how to glue Fortran and C/C++ with iso_c_binding.
  - Learn how to build mixed-language projects with CMake.
keypoints:
  - iso_c_binding is the standard and recommended way to glue Fortran and C/C++ in a portable way.
---

## Learning goals

- Calling Fortran libraries from C(++)
- Calling C(++) libraries from Fortran

---

## Example

Imagine you are on a desert island and wish to compute $\pi$.
You have a computer with you with compilers installed but no math libraries and no Wikipedia.

**SPOILER BELOW**

<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>

Here is one way of doing it - throwing darts (generating random points in an
interval and checking whether the points are within a given radius):

![]({{ site.baseurl }}/img/darts.svg)

Discuss briefly how you can approximate $\pi$ from such an experiment and how you would program it.

In this workshop we will implement this example in 3 different languages (C++,
Fortran, Python) and demonstrate how to call this functionality across
languages.

Later we will combine these 3 implementations in a Python package and learn few useful tricks.

---

## Questions

- Have you ever tried to call C functions from Fortran or vice versa?
- What were the problems?
- What is name mangling? Related question: How does [GitHub](https://github.com) avoid name collisions between projects?
- How have you worked around the problem of name mangling?


### Problems before `iso_c_binding` arrived

- Codes needed to work around name mangling.
- Codes needed to figure out bit representations of data types.

---

## Exercise 1: testing the algorithm

Before you continue, create a fresh directory for this session.

Test the following Python code (Python newcomers please sit next to an experienced Python developer):

```python
import random


def _distance_to_origin_squared(x, y):
    return x*x + y*y


def approximate_pi(num_points):
    random.seed(0)

    num_inside = 0
    for _ in range(num_points):
        x = random.uniform(0.0, 1.0)
        y = random.uniform(0.0, 1.0)
        if _distance_to_origin_squared(x, y) < 1.0:
            num_inside += 1

    # we multiply by 4 to get the full circle
    # from the 4 segments
    return 4.0*num_inside/float(num_points)
```

### Questions

- Does it converge to the right number?
- Does it take longer when you increase `num_points`?
- Why is there no `math.sqrt` in the code?

---

## Exercise 2: testing the isolated C++ and Fortran implementations

Let us now fetch an example from the web:

```shell
$ git clone --branch exercise/cxx-fortran https://github.com/bast/python-cffi-demo.git cxx-fortran
```

Let us inspect the file structure:

```shell
$ cd cxx-fortran
```

We see the following files:

```
.
|-- CMakeLists.txt
`-- island
    |-- main.cpp
    |-- main.f90
    |-- pi.cpp
    |-- pi.f90
    |-- pi.h
    `-- pi.py
```

Your task is to configure and build the code with:

```shell
$ mkdir build
$ cd build
$ cmake ..
$ make
```

- Test the binaries built under `build/bin/`.
- Also have a look at [CMakeLists.txt](https://github.com/bast/python-cffi-demo/blob/exercise/cxx-fortran/CMakeLists.txt) and see if you can make sense of it.

---

## Exercise 3: C++ and Fortran talking to each other

Your task now is to let C++ also call the Fortran implementation and to let
Fortran also call the C++ implementation.

For this first uncomment the two following lines:

- [island/main.f90](https://github.com/bast/python-cffi-demo/blob/exercise/cxx-fortran/island/main.f90#L17)
- [island/main.cpp](https://github.com/bast/python-cffi-demo/blob/exercise/cxx-fortran/island/main.cpp#L8)

Then try to recompile - you will observe that the code has unmet dependencies.
Try to fix these (use out-commented code for hints).
Finally, verify that the binaries work and that they indeed call both implementations.

Discuss with the group how it works.


### Questions

- Why did we need to do more work on the Fortran side?
- How does C++ know the type information?
