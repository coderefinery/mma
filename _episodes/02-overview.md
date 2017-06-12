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

In this episode we will briefly highlight the pros and cons of the Cython, and show how the same source code can be used SWIG and Boost. In the next section we
will go more in detail on how to use Pybind11.


**Cython**: "All of this makes Cython the ideal language for wrapping external C
libraries, embedding CPython into existing applications, and for fast C modules
that speed up the execution of Python code.", see webpage http://cython.org

"**Pybind11** is a lightweight-header only library that exposes C++-types in Python
and vice versa, mainly to create Python bindings of existing C++ code.",
according to the Pybind11 web page,
https://pybind11.readthedocs.io/en/stable/intro.html. The point it is
lightweight and targeting the combination of Python and C++11compliant
compilers. Consequently, the interface code becomes more straight forward.

**SWIG** (Simplified Wrapper and Interface Generator) is a tool that simplifies the
two step process of making a wrapper and generating a interface which makes the
wrapper callable from the interpreter. According to the SWIG documentation,
"SWIG was originally designed to make it extremely easy for scientist and
engineers to build extensible scientific software without having a degree in
software engineering". So SWIG should really be the only thing we need, right?
Could be, but before giving a motivation for the other tools, it is worth
mentioning that SWIG support a range of interpreting languages (C#,Common Lisp,
Go,R, Lua ...), not only Python.

**Boost** is a huge C++-library which works with almost any C++-compiler. The
Python interface tool was added to the Boost library around 2002 by David
Abrahams. Hence, if you a have special C++-compiler, Boost.Python could be your
most suitable tool for integrating Python and C++.

**F2PY** is a tool for interfacing Fortan and Python. According to "Python
Scripting for Computational Science" transferring Numpy arrays between Python
and compiled Fortran code is easier with F2PY than SWIG.

In the following we will show how to C++-functions can be enabled for use with
Python by using Cython and Pybind11. The text also demonstrates how the same
source code can be Python enabled by using SWIG or Boost. You may read these
parts after the Coderefinery workshop.

### The source code

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
approximations of trigonometric functions sine and cosine implemented
as taylor series. Their arguments are the angle and the number of terms in
the taylor series. In the example only `ts_sin()` and `ts_cos()` will be
callable from the python interpreter. Though, these functions will be
renamed as `sin()` and `cos()`, taking two arguments.

### Cython

There is several ways to a library with the use of Cython. We will demonstrate
how to do it with CMake. The other prefered way would be with the use of
distutils, which is also described, but we will not demonstrate it as part of
the workshop.

You will need to clone a git repository for having all the necessary files
for the demonstratino

```bash
$ git clone https://github.com/blindij/python-ctools-demo.git
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
Firste we will look at the `taylor.pyx`. The expressions in this file are very
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

This is how the CMake files looks like:

```cmake
cmake_minimum_required(VERSION 2.8 FATAL_ERROR)
project(taylor)

SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY
   ${CMAKE_BINARY_DIR}/lib
   )
 
# Detect and activate pybind11. Stop if not is not found
find_package(pybind11 REQUIRED)

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

### SWIG

Our source code contains three functions, Taylor series of sin(), cos() and a
helper function factorial(). We will make sin() and cos() available in our
Python interpreter with the use of SWIG

Start out with at empty subdirectory, your C++ compiler and the Anaconda2
environment available in your path. Activate the swig-example environment:
```bash
[lynx@~]$ mkdir swig
[lynx@swig]$ type conda
conda is /home/lynx/anaconda2/bin/conda
[lynx@swig]$ type g++
g++ is /share/apps/software/Core/GCCcore/6.3.0/bin/g++
[lynx@swig]$ g++ --version
g++ (GCC) 6.3.0
Copyright (C) 2016 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
[lynx@swig]$ source activate swig-example
(swig-example)[lynx@swig]$
```
Now we create a src subdirectory with the source files taylor_series.h and
taylor_series.h

Here is the contents of the taylor_series.h:
```cpp
// taylor_series.h
#ifndef TAYLOR_SERIES_H_
#define TAYLOR_SERIES_H_

extern unsigned long long factorial( int n);
extern double sin(double x, int N);
extern double cos(double x,int N);

#endif // TAYLOR_SERIES_H_
```

The contents of taylor_series.cpp:

```cpp
// taylor_series.cpp
#include <math.h>
#include "taylor_series.h"

unsigned long long factorial( int n){
  unsigned long long result = 1;
  if ( n != 0)
     for (int i = 0; i < n; i++)
       result = result * (i+1);
  return result;
};

double sin(double x, int N){
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

double cos(double x,int N) {
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
Now we need a interface file which describes how these functions can be called.
In SWIG nomenclature this is a i-file. Here we define the module name and which
functions that needs wrappers.

```cpp
// taylor.i
%module taylor
%{
  // include C++ header
#include "taylor_series.h"
  %}

double sin(double x, int N);
double cos(double x, int N);
```
The module will be named taylor and it is the function sin() and cos() which
will be available for Python. We leave out the function factorial().

We generate the wrapper, and compile the code to a share library called
_taylor.so.

```shell
(swig-example) [lynx@swig]$ swig -python taylor.i
(swig-example) [lynx@swig]$ g++ -c -fpic -I. `python-config --cflags` src/taylor_series.cpp taylor_wrap.c
(swig-example) [lynx@swig]$ g++ -shared `python-config --ldflags` taylor_wrap.o taylor_series.o -o _taylor.so
(swig-example) [lynx@swig]$ ls
src       taylor.py   taylor_series.o  taylor_wrap.c
taylor.i  taylor.pyc  _taylor.so          taylor_wrap.o
(swig-example) [lynx@swig]$ python
```

There several files in our subdirectory. It is the _taylor.so we will load into
our Python interpreter. Start python as above.

```python
>>> import taylor
>>> taylor.sin(3.141592653/4,14)
0.7071067810822858
>>> taylor.cos(3.141592653/4,14)
0.7071067812908091
>>> import scipy
>>> scipy.sin(3.141592653/4)
0.70710678108228586
>>> scipy.cos(3.141592653/4)
0.70710678129080917
```
Our Taylor-functions are available for use by Python.

There is a jungle of C++ constructs and complexities which we have avoided in
this example. For instance, the functions arguments are passed by value, which
not very likely in C++, as arguments are passed by reference.

### Arguments passed by reference
As we want to use functions which call by reference, our interface file becomes less intuitive.

Here is our C++-source code of the Taylor Series, the taylor_series.cpp:
```cpp
//
// taylor_series.cpp
//
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
  long double numerator,denomi;
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
  long double numerator,denomi;
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
The file tayour_series.h:

```cpp
#ifndef TAYLOR_SERIES_H_
#define TAYLOR_SERIES_H_

extern unsigned long long factorial( int n);
extern double ts_sin(double& x, int N);
extern double ts_cos(double& x,int N);

#endif // TAYLOR_SERIES_H_
```

To make these functions available for the Python interpreter with the use of
SWIG, we need to write a interface file, a %.i file. In our example called
tss.i:
```cpp
// file: tss.i
%module tss
%{
  // include C++ header
#include "taylor_series.h"
  %}

%include "typemaps.i"
%apply double *INPUT {double& x }
%include "taylor_series.h"
```




### Boost
Make a new subdirectory with a src subdirectory:

```shell
(boost-example) [lynx@boost]$ mkdir -p boost/src
(boost-example) [lynx@boost]$ cd boost
```
In the src-directory make a file boost_taylor.cpp with following contains:
```cpp
#include <boost/python/module.hpp>
#include <boost/python/def.hpp>
#include <boost/python/tuple.hpp>

unsigned long long factorial( int n){
  unsigned long long result = 1;
  if ( n != 0)
     for (int i = 0; i < n; i++)
       result = result * (i+1);
  return result;
};

double sin(double x, int N){
  long double numerator,denomi;
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

double cos(double x,int N) {
  long double numerator,denomi;
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


BOOST_PYTHON_MODULE(taylor_boost)
{
  using namespace boost::python;
  def("sin", sin);
  def("cos", cos);
}

```
Compiling the boost version and start python:

```shell
g++ --std=c++11 --shared src/boost_taylor.cpp -I${HOME}/anaconda2/envs/boost-example/include `python-config --cflags` -L${HOME}/anaconda2/envs/boost-example/lib/ -lboost_python `python-config --ldflags` -fpic -o taylor_boost.so
python
```
Load the module:
```python
ython 2.7.13 |Continuum Analytics, Inc.| (default, Dec 20 2016, 23:09:15)
[GCC 4.4.7 20120313 (Red Hat 4.4.7-1)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
Anaconda is brought to you by Continuum Analytics.
Please check out: http://continuum.io/thanks and https://anaconda.org
>>> import taylor_boost
>>> taylor_boost.sin(3.141592653/3,15)
0.8660254036861398
>>> exit()
```
Remember to deactivate the environment:

```shell
source deactivate
```

