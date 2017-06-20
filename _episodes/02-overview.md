---
layout: episode
title: Overview over different approaches
teaching: 10
exercises: 0
questions:
  - Which tools are out there?
objectives:
  - Get a birds-eye overview of existing technologies.
keypoints:
  - Many tools exist with different scope, expressiveness, and complexity.
---

## Overview

There exist several technologies which make it possible to couple Python and compiled libraries:

- Cython
- Pybind11
- SWIG
- Boost
- F2PY
- CFFI

**Cython**: "Cython’s power comes from the way it combines Python and C: 
it feels like Python while providing easy access to C. Cython is situated 
between high-level Python and low-level C; one might call it a creole 
programming language." [Cython](http://shop.oreilly.com/product/0636920033431.do) by Kurt Smith

"**Pybind11** is a lightweight-header only library that exposes C++-types in Python
and vice versa, mainly to create Python bindings of existing C++ code.",
according to the Pybind11 [web page](https://pybind11.readthedocs.io/en/stable/intro.html).
The point it is lightweight and targeting the combination of Python and C++11
compliant compilers. Consequently, the interface code becomes more straight
forward.

**SWIG** (Simplified Wrapper and Interface Generator) is a tool that simplifies the
two step process of making a wrapper and generating a interface which makes the
wrapper callable from the interpreter. According to the SWIG documentation,
"SWIG was originally designed to make it extremely easy for scientist and
engineers to build extensible scientific software without having a degree in
software engineering". SWIG support a range of interpreting languages
(C#,Common Lisp, Go,R, Lua ...), not only Python.

**Boost** is a huge C++-library which works with almost any C++-compiler. The
Python interface tool was added to the Boost library around 2002 by David
Abrahams. Hence, if you a have special C++-compiler, Boost.Python could be your
most suitable tool for integrating Python and C++.

**F2PY** is a tool for interfacing Fortran and Python. According to "Python
Scripting for Computational Science" transferring Numpy arrays between Python
and compiled Fortran code is easier with F2PY than SWIG.

**CFFI** C Foreign Function Inteface for Python requires only knowledge of C
and Python. There is no need to gain knowledge of a domain specific language.

### How too choose the most suitable tool?
Here are our rule of thumb:

If you need to interface C/C++, four of the mentioned tools/libraries are
available to you. Are you most experienced with python and python is your
preferred language, choose Cython. If your competence are mainly in C++,
but you need to provide a python library, choose Pybind11.

If python is not your choice, but you need to bind another scripting language
with compiled code, then go with SWIG, but use CMake to hide complexity in the
building process.

If C or Fortran is your preferred compiled language, then consider CFFI.

### Combining a C++-code with Cython or Pybind11

In the following we will show how C++-functions can be enabled for use with
Python by using Cython and Pybind11.

#### The source code

Below is the source code we will use, `taylor_series.h` and `taylor_series.cpp`:

```cpp
// file: taylor_series.h
#ifndef TAYLOR_SERIES_H_
#define TAYLOR_SERIES_H_

unsigned long long factorial( int n);
double ts_sin(double& x, int N);
double ts_cos(double& x, int N);

#endif // TAYLOR_SERIES_H_
```

```cpp
// file: taylor_series.cpp
#include <iostream>
#include <math.h>
#include "taylor_series.h"

unsigned long long factorial( int n){
  unsigned long long result = 1;
  if ( n != 0)
     for (int i = 0; i < n; i++)
       result = result * (i+1);
  return result;
};

double ts_sin(double& x, int N){
  double    sum = 0.0;
  int par,sign;
  unsigned long int fac;
  for (int i = 0; i < N; i++) {
    par = (1+2*i);
    fac = factorial(par);
    sign = pow((-1),i);
    sum = sum + sign*pow(x,par)/fac;

  }

  return sum;
}

double ts_cos(double& x,int N) {
  double sum = 0.0;
  int par,sign;
  unsigned long int fac;
  for (int i = 0; i < N; i++) {
    par = 2*i;
    fac = factorial(par);
    sign = pow((-1),i);
    sum = sum + sign*pow(x,par)/fac;
  }
  return sum;
}
```

The source code implements three functions `factorial()`, `ts_sin()` and
`ts_cos()`. `factorial()` returns the factorial of an integer N.
N is given as an argument to the function. `ts_sin()` and `ts_cos()` are
approximations of the trigonometric functions sine and cosine implemented
as taylor series. Their arguments are the angle and the number of terms in
the taylor series. In the example only `ts_sin()` and `ts_cos()` will be
callable from the python interpreter. Though, these functions will be
renamed on the Python side as `sin()` and `cos()`, taking two arguments.

### Cython

There is several ways to build a library with the use of Cython. We will
demonstrate how to do it with CMake. The other preferred way would be with the
use of `distutils`. How to build with `distutils` is also shown be low.

You will need to clone a git repository:

```bash
$ git clone https://github.com/coderefinery/python-ctools-demo.git --recursive
$ cd python-ctools-demo/cmake-demo
```

#### Building a library with Cython and CMake

In the cmake-demo subdirectory you will find these files:

```shell
.
├── cmake
│   ├── FindCython.cmake
│   ├── ReplicatePythonSourceTree.cmake
│   └── UseCython.cmake
├── CMakeLists.txt
└── src
    ├── CMakeLists.txt
    ├── taylor.pyx
    ├── taylor_series.cpp
    └── taylor_series.h

2 directories, 8 files
```

First we will look at the `taylor.pyx`. The expressions in this file are very
python-like:

```python
cdef extern from "taylor_series.h":
     double ts_sin(double& x, int N)
     double ts_cos(double& x, int N)

def sin(double x, int N):
    return ts_sin(x,N)

def cos(double x, int N):
    return ts_cos(x,N)   
```
Cython provides the glue between Python and C++. It will wrap our source code
in way such that it is callable from Python. However, we will need to state
how we will call the C++-functions. This is the purpose of the `taylor.pyx`.

The `CMakeLists.txt` describes the building process

```cmake
cmake_minimum_required(VERSION 2.8 FATAL_ERROR)
project(taylor)

SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY
   ${CMAKE_BINARY_DIR}/lib
   )
 
# Add the cmake subdirectory to the module path
set(CMAKE_MODULE_PATH
	${CMAKE_MODULE_PATH}
	${PROJECT_SOURCE_DIR}/cmake
	)
# Detects and activates Cython
include(UseCython)

add_subdirectory(src)
```

We will build a library and it will be place in the lib subdirectory in the
build directory. The cmake subdirectory is added to the CMake module path.
The cmake subdirectory contains files from the [cython-cmake-example](https://github.com/thewtex/cython-cmake-example).

This subdirectory is added because the CMake functions these files provides is
needed in the building process. Some of you may have these files installed, others may not.

In `CMakeLists.txt` in the src subdirectory the name of our library is written
as the first argument to `cython_add_module()`. The other arguments are the
source files the library depends upon.

```cmake
#include the current directory in search for include files
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

# Specifies that Cython source files should generate C++
set_source_files_properties(taylor.pyx
  PROPERTIES
  CYTHON_IS_CXX TRUE)

# Adds and compiles Cython into an extension module
cython_add_module(taylor taylor.pyx taylor_series.cpp)
```

The building process is a straight forward CMake building process:

```shell
$ mkdir build
$ cd build
$ cmake ..
$ make
$ cd lib
$ python
>>> import taylor
>>> taylor.cos(3.1415926535/3,15)
```

Here is the output from a Linux build and the following python session:
```shell
(cython-example) [lynx@lille-login2 cmake-demo]$ mkdir build
(cython-example) [lynx@lille-login2 cmake-demo]$ cd build
(cython-example) [lynx@lille-login2 build]$ cmake ..
-- The C compiler identification is GNU 4.8.5
-- The CXX compiler identification is GNU 4.8.5
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Found PythonInterp: /home/lynx/anaconda2/envs/cython-example/bin/python (found version "2.7.13") 
-- Found Cython: /home/lynx/anaconda2/envs/cython-example/bin/cython  
-- Found PythonLibs: /usr/lib64/libpython2.7.so (found version "2.7.5") 
-- Configuring done
-- Generating done
-- Build files have been written to: /home/lynx/tmp/python-ctools-demo/cython-demo/cmake-demo/build
(cython-example) [lynx@lille-login2 build]$ make
[ 33%] Compiling Cython CXX source for taylor...
Scanning dependencies of target taylor
[ 66%] Building CXX object src/CMakeFiles/taylor.dir/taylor.cxx.o
[100%] Building CXX object src/CMakeFiles/taylor.dir/taylor_series.cpp.o
Linking CXX shared module ../lib/taylor.so
[100%] Built target taylor
(cython-example) [lynx@lille-login2 build]$ cd lib
(cython-example) [lynx@lille-login2 lib]$ python
Python 2.7.13 |Continuum Analytics, Inc.| (default, Dec 20 2016, 23:09:15) 
[GCC 4.4.7 20120313 (Red Hat 4.4.7-1)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
Anaconda is brought to you by Continuum Analytics.
Please check out: http://continuum.io/thanks and https://anaconda.org
>>> import taylor
>>> taylor.cos(3.1415926535/3,15)
0.500000000025921
>>> 
```

#### Building the library using `distutils`

```bash
$ cd ../../../distutils-demo/
```
This chapter shows how we can use `distutils` to build the taylor library.
Change to the distutils-demo subdirectory. This subdirectory looks like this:

```bash
.
├── setup.py
└── src
    ├── taylor.pyx
    ├── taylor_series.cpp
    └── taylor_series.h

1 directory, 4 files
```
The contents of the files in the `src` subdirectory are equal to the CMake example.
Instead of the CMake files, we find a `setup.py`. Here is the contents
of the `setup.py`:

```python
from distutils.core import setup, Extension
from Cython.Build import cythonize

ext = Extension("taylor",
                sources=["src/taylor.pyx","src/taylor_series.cpp"],
                language="c++")

setup(
	name = "taylor_series",
	ext_modules = cythonize([ext])
    )
```

We name the library `taylor` and state that the library depends on two source
files. C++ is the compile language which will be used.

To build and load the library we execute python:

```bash
$ python setup.py build_ext -i
$ python
>>> import taylor
>>> taylor.cos(3.1415926535/3,15)

```

Here is output from the building process on a Linux system:

```bash
(cython-example) [lynx@lille-login2 distutils-demo]$ python setup.py build_ext -i
running build_ext
building 'taylor' extension
creating build
creating build/temp.linux-x86_64-2.7
creating build/temp.linux-x86_64-2.7/src
gcc -pthread -fno-strict-aliasing -g -O2 -DNDEBUG -g -fwrapv -O3 -Wall -Wstrict-prototypes -fPIC -I/home/lynx/anaconda2/envs/cython-example/include/python2.7 -c src/taylor.cpp -o build/temp.linux-x86_64-2.7/src/taylor.o
cc1plus: warning: command line option ‘-Wstrict-prototypes’ is valid for C/ObjC but not for C++ [enabled by default]
gcc -pthread -fno-strict-aliasing -g -O2 -DNDEBUG -g -fwrapv -O3 -Wall -Wstrict-prototypes -fPIC -I/home/lynx/anaconda2/envs/cython-example/include/python2.7 -c src/taylor_series.cpp -o build/temp.linux-x86_64-2.7/src/taylor_series.o
cc1plus: warning: command line option ‘-Wstrict-prototypes’ is valid for C/ObjC but not for C++ [enabled by default]
g++ -pthread -shared -L/home/lynx/anaconda2/envs/cython-example/lib -Wl,-rpath=/home/lynx/anaconda2/envs/cython-example/lib,--no-as-needed build/temp.linux-x86_64-2.7/src/taylor.o build/temp.linux-x86_64-2.7/src/taylor_series.o -L/home/lynx/anaconda2/envs/cython-example/lib -lpython2.7 -o /home/lynx/tmp/python-ctools-demo/cython-demo/distutils-demo/taylor.so
(cython-example) [lynx@lille-login2 distutils-demo]$ python
Python 2.7.13 |Continuum Analytics, Inc.| (default, Dec 20 2016, 23:09:15) 
[GCC 4.4.7 20120313 (Red Hat 4.4.7-1)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
Anaconda is brought to you by Continuum Analytics.
Please check out: http://continuum.io/thanks and https://anaconda.org
>>> import taylor
>>> taylor.cos(3.1415926535/3,15)
0.500000000025921
>>> 
```

### PyBind11

In the cloned source tree (`git clone https://github.com/blindij/python-ctools-demo.git`)
, change to the pybind11-demo subdirectory. Here you find these files:

```shell
.
├── CMakeLists.txt
├── pybind11
│   ├── CMakeLists.txt

....

└── src
    ├── CMakeLists.txt
    ├── py11taylor.cpp
    ├── taylor_series.cpp
    └── taylor_series.h

1 directory, 5 files
```
The Pybind11 file:

```C++
// py11taylor.cpp
#include <pybind11/pybind11.h>
#include "taylor_series.h"

namespace py = pybind11;

PYBIND11_PLUGIN(taylor) {
  py::module t("taylor", "Taylor Series tailored plugin");
   t.def("sin", &ts_sin, "A sinus Taylor Series function")
    .def("cos", &ts_cos, "A cosinus Taylor Series function");
  return t.ptr();
}
```

This is how the CMake files look like:

```cmake
cmake_minimum_required(VERSION 2.8 FATAL_ERROR)
project(taylor)

SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY
   ${CMAKE_BINARY_DIR}/lib
   )

# The pybind11 is the Pybind11 project added as  submodule
# This CMakeLists.txt in the pybind11 directory
# enables pybind11  for use with python

add_subdirectory(pybind11)

# Alternative if Pybind11 is part of your Python installation
# Detect and activate pybind11. Stop if not is not found
# find_package(pybind11 REQUIRED)

add_subdirectory(src)
```

`src`-subdirectory:
 
```cmake
#include the current directory in search for include files
include_directories(${CMAKE_CURRENT_SOURCE_DIR})

# Adds and compiles Cython into an extension module
pybind11_add_module(taylor py11taylor.cpp taylor_series.cpp)
```

To build:

```bash
$ mkdir build
$ cd build
$ cmake ..
$ cd lib
$ python
>>> import taylor
>>> taylor.cos(3.141592653/3,15)
```

Output from building on Linux with Anaconda2 extended with Pybind11:

```bash
(pybind11-example) [lynx@lille-login2 pybind11-demo]$ mkdir build
(pybind11-example) [lynx@lille-login2 pybind11-demo]$ cd build
(pybind11-example) [lynx@lille-login2 build]$ cmake ..
-- The C compiler identification is GNU 4.8.5
-- The CXX compiler identification is GNU 4.8.5
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Found PythonInterp: /home/lynx/anaconda2/envs/pybind11-example/bin/python (found version "2.7.13") 
-- Found PythonLibs: /home/lynx/anaconda2/envs/pybind11-example/lib/libpython2.7.so
-- Performing Test HAS_CPP14_FLAG
-- Performing Test HAS_CPP14_FLAG - Failed
-- Performing Test HAS_CPP11_FLAG
-- Performing Test HAS_CPP11_FLAG - Success
-- Performing Test HAS_FLTO
-- Performing Test HAS_FLTO - Success
-- LTO enabled
-- Configuring done
-- Generating done
-- Build files have been written to: /home/lynx/tmp/python-ctools-demo/pybind11-demo/build
(pybind11-example) [lynx@lille-login2 build]$ make
Scanning dependencies of target taylor
[ 50%] Building CXX object src/CMakeFiles/taylor.dir/py11taylor.cpp.o
[100%] Building CXX object src/CMakeFiles/taylor.dir/taylor_series.cpp.o
Linking CXX shared module ../lib/taylor.so
[100%] Built target taylor
(pybind11-example) [lynx@lille-login2 build]$ cd lib
(pybind11-example) [lynx@lille-login2 lib]$ python
Python 2.7.13 |Continuum Analytics, Inc.| (default, Dec 20 2016, 23:09:15) 
[GCC 4.4.7 20120313 (Red Hat 4.4.7-1)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
Anaconda is brought to you by Continuum Analytics.
Please check out: http://continuum.io/thanks and https://anaconda.org
>>> import taylor
>>> taylor.cos(3.141592653/3,15)
0.5000000001702586
>>> 
```
## Git submodule
In the pybind11 example we are depending upon the Pybind11 git repository.
It is cloned as part of a recursive clone. The pybind11 repository is marked as
a submodule. Pybind11 was added to the `python-ctools-demo` repository in the
following way (the current directory being `python-ctools-demo`):

```bash
$ git submodule add  https://github.com/pybind/pybind11.git
$ git commit -am "Added Pybind11 module"
```

When a repository contains one or several submodules, you will need to add
`--recursive` as an argument to `git clone`. Otherwise the submodule will
not be cloned, and the submodule will just be an empty directory.

An alternative is to initialize and update the submodule, specifically.

```bash
$ git clone https://github.com/blindij/python-ctools-demo.git
$ cd python-ctools-demo/
$ git submodule init
$ git submodule update
```

Here is the output from a Linux session:

```bash
[lynx@lille-login2 tmp]$ git clone https://github.com/blindij/python-ctools-demo.git
Cloning into 'python-ctools-demo'...
remote: Counting objects: 81, done.
remote: Compressing objects: 100% (67/67), done.
remote: Total 81 (delta 18), reused 73 (delta 10), pack-reused 0
Unpacking objects: 100% (81/81), done.
[lynx@lille-login2 tmp]$ cd python-ctools-demo/
[lynx@lille-login2 python-ctools-demo]$ git submodule init
Submodule 'phoneticA/pybind11' (https://github.com/pybind/pybind11.git) registered for path 'phoneticA/pybind11'
Submodule 'pybind11-demo/pybind11' (https://github.com/pybind/pybind11.git) registered for path 'pybind11-demo/pybind11'
[lynx@lille-login2 python-ctools-demo]$ git submodule update
Cloning into 'pybind11-demo/pybind11'...
remote: Counting objects: 8182, done.
remote: Compressing objects: 100% (17/17), done.
remote: Total 8182 (delta 4), reused 11 (delta 3), pack-reused 8161
Receiving objects: 100% (8182/8182), 2.85 MiB | 1.12 MiB/s, done.
Resolving deltas: 100% (5444/5444), done.
Submodule path 'pybind11-demo/pybind11': checked out '13d8cd2cc7566de34d724f428ea7a6b6448d6a0c'

```