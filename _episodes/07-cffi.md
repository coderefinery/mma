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
  - Big advantage of CFFI is that the two sides do not need to adapt to each other.
---

## Setting the stage

We will stay with the same pi example but please picture your project instead.

Imagine one of two situations:

- Either you have a Python code and want to create a C/C++/Fortran back-end.
- You have a (possibly legacy) C/C++/Fortran code and wish to create a Python front-end.

---

## Learning goals

- [Wrap with Python](https://twitter.com/ThePracticalDev/status/823305296521154563)


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

```shell
$ cd build
$ make

Scanning dependencies of target pi_fortran
[ 12%] Building Fortran object CMakeFiles/pi_fortran.dir/pi/pi.f90.o
[ 25%] Linking Fortran shared library lib/libpi_fortran.so
[ 25%] Built target pi_fortran
Scanning dependencies of target pi_cpp
[ 37%] Building CXX object CMakeFiles/pi_cpp.dir/pi/pi.cpp.o
[ 50%] Linking CXX shared library lib/libpi_cpp.so
[ 50%] Built target pi_cpp
Scanning dependencies of target pi_cpp.x
[ 62%] Building CXX object CMakeFiles/pi_cpp.x.dir/pi/main.cpp.o
[ 75%] Linking CXX executable bin/pi_cpp.x
[ 75%] Built target pi_cpp.x
Scanning dependencies of target pi_fortran.x
[ 87%] Building Fortran object CMakeFiles/pi_fortran.x.dir/pi/main.f90.o
[100%] Linking Fortran executable bin/pi_fortran.x
[100%] Built target pi_fortran.x
```

After that we go one level up and enter the `pi` directory:

```shell
$ cd ..
$ cd pi
$ ls -l

total 32
-rw------- 1 bast users 211 May 28 00:51 main.cpp
-rw------- 1 bast users 542 May 28 00:51 main.f90
-rw------- 1 bast users 579 May 28 00:51 pi.cpp
-rw------- 1 bast users 985 May 28 00:51 pi.f90
-rw------- 1 bast users 387 May 28 00:51 pi.h
-rw------- 1 bast users 452 May 28 00:51 pi.py
```

And we fetch two files from the web:

```shell
$ wget https://raw.githubusercontent.com/bast/python-cffi-demo/master/pi/cffi_helpers.py
$ wget https://raw.githubusercontent.com/bast/python-cffi-demo/master/pi/__init__.py
```

The first file contains a function `get_lib_handle` tells CFFI where to find the header file and the
dynamic library and from this CFFI will create a Python interface.

We have places this function into a separate file so that you can reuse it for
different libraries.

The second file, `__init__.py` is the package interface file which exposes 3
functions:

```python
__all__ = [
    'approximate_pi_python',
    'approximate_pi_c',
    'approximate_pi_fortran',
]
```

With these two files we have created a Python interface!

Let us first test it (at this point you need the `cffi` package activated):

```shell
$ PI_BUILD_DIR=build python

Python 3.6.1 (default, Mar 27 2017, 00:27:06)
[GCC 6.3.1 20170306] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import pi
>>> print(pi.approximate_pi_c(100))
3.12
>>>
```

Why do we need to set `PI_BUILD_DIR` when importing our `pi` package?

Now we can run some timings - create a file called `test.py` which contains:

```python
import time
import pi

num_points = 2000000

def print_timings():
    print('num points: {0}'.format(num_points))

    for (lang, function) in [('python', pi.approximate_pi_python),
                             ('c', pi.approximate_pi_c),
                             ('fortran', pi.approximate_pi_fortran)]:
        t0 = time.clock()
        result = function(num_points)
        time_spent = time.clock() - t0

        print('{0:7s} pi={1:.5f} time spent: {2:.3f} sec'.format(lang, result, time_spent))

if __name__ == '__main__':
    print_timings()
```

Try it out:

```shell
$ PI_BUILD_DIR=build python test.py
```

Experiment with varying number of points.

After testing the interface, take the time to study the files and discuss the code.

---

## Exercise 5: adding automated testing

### Motivations for testing your C/C++/Fortran code with Python

- Forces you to create a clean interface (good)
- Nice byproduct: you have a Python interface (good)
- Encourages dynamic library (good)
- You can write and prototype tests without recompiling/relinking the library (good)
- Allows you to use the wonderfully lightweight [pytest](http://pytest.org) (no more excuses for the Fortran crowd)


### How?

We will use [pytest](https://docs.pytest.org) for its simplicity.

For this you can enhance the `test.py` script with few test functions:
[https://github.com/bast/python-cffi-demo/blob/master/test.py](https://github.com/bast/python-cffi-demo/blob/master/test.py)

Run the test set with:

```shell
$ PI_BUILD_DIR=build pytest -vv test.py
```

And you will hopefully see:

```shell
==================================================== test session starts =====================================================
platform linux -- Python 3.6.1, pytest-3.1.0, py-1.4.33, pluggy-0.4.0 -- /home/bast/mma/python-cffi-demo/venv/bin/python3
cachedir: .cache
rootdir: /home/bast/python-cffi-demo, inifile:
collected 3 items

test.py::test_pi_python PASSED
test.py::test_pi_c PASSED
test.py::test_pi_fortran PASSED

================================================== 3 passed in 5.33 seconds ==================================================
```

If you use [GitHub](https://github.com), you can go one step further:

-  Put this project on GitHub.
-  Log into [Travis CI](https://travis-ci.org) with your GitHub account.
-  Enable testing for this project.
-  Add a [.travis.yml](https://github.com/bast/python-cffi-demo/blob/master/.travis.yml) file.

Then each commit gets automatically tested and you get a [build and test
history](https://travis-ci.org/bast/python-cffi-demo/builds) for your project.

---

## Exercise 6: adding a setup script

Finally we want to put a cherry on top of our project and make it possible to
install with `pip` and even upload to [PyPI - the Python Package Index](https://pypi.python.org/pypi).

For this go to the root directory of your project (one level above the `pi`
directory) and fetch a setup script which we have already prepared for you:

```shell
$ wget https://raw.githubusercontent.com/bast/python-cffi-demo/master/setup.py
```

We will inspect it in a minute but let us first try it out:

- Open a new terminal.
- Activate a new virtual environment:

```shell
$ cd /tmp
$ virtualenv venv
$ source venv/bin/activate
$ pip install /path/to/the/project
```

Here is how it looks on my machine:

```shell
$ pip install /home/bast/python-cffi-demo

Processing /home/bast/python-cffi-demo
Collecting cffi (from pi==0.0.0)
  Using cached cffi-1.10.0-cp36-cp36m-manylinux1_x86_64.whl
Collecting pycparser (from cffi->pi==0.0.0)
Building wheels for collected packages: pi
  Running setup.py bdist_wheel for pi ... done
Successfully built pi
Installing collected packages: pycparser, cffi, pi
Successfully installed cffi-1.10.0 pi-0.0.0 pycparser-2.17
```

In the setup script we subclass the `install` and `build` methods and call
CMake under the hood which configures, builds, and installs the libraries in
the right place.

If you are not ready for [PyPI](https://pypi.python.org/pypi)
yet, you can also install directly from GitHub:

```shell
$ pip install git+https://github.com/bast/python-cffi-demo.git
$ python -c 'import pi; print(pi.approximate_pi_c(100))'
```
