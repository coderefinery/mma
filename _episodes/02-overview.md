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

- SWIG
- Boost
- F2PY
- Cython
- Pybind11

In this episode we will briefly highlight the pros and cons of the first four and then
spend some more time with Pybind11 in the next section.

Simplified Wrapper and Interface Generator (**SWIG**) is a tool that simplifies the
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

**Cython**: "All of this makes Cython the ideal language for wrapping external C
libraries, embedding CPython into existing applications, and for fast C modules
that speed up the execution of Python code.", see webpage http://cython.org

"**Pybind11** is a lightweight-header only library that exposes C++-types in Python
and vice versa, mainly to create Python bindings of existing C++ code.",
according to the Pybind11 web page,
https://pybind11.readthedocs.io/en/stable/intro.html. The point it is
lightweight and targeting the combination of Python and C++11compliant
compilers. Consequently, the interface code becomes more straight forward.


### SWIG

Our source code contains three functions, Taylor series of sin(), cos() and a
helper function factorial(). We will make sin() and cos() available in our
Python interpreter with the use of SWIG

Start out with at empty subdirectory, your C++ compiler and the Anaconda2
environment available in your path. Activate the swig-example environment:
```shell
[lynx@~]$mkdir swig
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
(swig-example) [lynx@swig]$ g++ -c -fpic -Isrc `python-config --cflags` src/taylor_series.cpp taylor_wrap.c
(swig-example) [lynx@swig]$ g++ -shared `python-config --ldflags` taylor_wrap.o taylor_series.o -o _taylor.so
(swig-example) [lynx@swig]$ ls
src       taylor.py   taylor_seriesv.o  taylor_wrap.c
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
As we want use functions which call by reference, our interface file becomes less intuitive.

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

### Cython
Make a subdirectory cython with an additional `src` subdirectory:

```shell
mkdir -p cython/src
cd cython/src
```

We will make directory structure with these files:

```shell
.
├── setup.py
└── src
    ├── taylor.pyx
    ├── taylor_series.cpp
    └── taylor_series.h

1 directory, 4 files
```

Let us start in the `src` subdirectory with the source files. cd into src,
if you have not done so and create `taylor_series.h` and `taylor_series.cpp`:

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

Cat the contents to the files, or create them in an editor:

```shell
(cython-example) [lynx@lille-login2 src]$ cat > taylor_series.h
Paste contents && press <Ctrl-d>
(cython-example) [lynx@lille-login2 src]$ cat > taylor_series.cpp
Paste contents && press <Ctrl-d>
```

The conents of the `taylor.pyx`:
```cpp
cdef extern from "taylor_series.h":
     double ts_sin(double& x, int N)
     double ts_cos(double& x, int N)

def sin(double x, int N):
    return ts_sin(x,N)

def cos(double x, int N):
    return ts_cos(x,N)
```

Create the `taylor.pyx` and cd up one level:

```shell
(cython-example) [lynx@lille-login2 src]$ cat > taylor_series.cpp
Paste contents && press <Ctrl-d>
(cython-example) [lynx@lille-login2 src]$ cd ..
(cython-example) [lynx@lille-login2 cython]$ 
```

We will make a `setup.py` for building the library. Here is the contents
of the file:

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

After we have created `setup.py`, we are ready to build the library:

```shell
(cython-example) [lynx@lille-login2 cython]$ cat > setup.py
Paste contents && press <Ctrl-d>
(cython-example) [lynx@lille-login2 cython]$ python setup.py build_ext -i
Compiling src/taylor.pyx because it changed.
[1/1] Cythonizing src/taylor.pyx
running build_ext
building 'taylor' extension
creating build
creating build/temp.linux-x86_64-2.7
creating build/temp.linux-x86_64-2.7/src
gcc -pthread -fno-strict-aliasing -g -O2 -DNDEBUG -g -fwrapv -O3 -Wall -Wstrict-prototypes -fPIC -I/home/lynx/anaconda2/envs/cython-example/include/python2.7 -c src/taylor.cpp -o build/temp.linux-x86_64-2.7/src/taylor.o
cc1plus: warning: command line option ‘-Wstrict-prototypes’ is valid for C/ObjC but not for C++ [enabled by default]
gcc -pthread -fno-strict-aliasing -g -O2 -DNDEBUG -g -fwrapv -O3 -Wall -Wstrict-prototypes -fPIC -I/home/lynx/anaconda2/envs/cython-example/include/python2.7 -c src/taylor_series.cpp -o build/temp.linux-x86_64-2.7/src/taylor_series.o
cc1plus: warning: command line option ‘-Wstrict-prototypes’ is valid for C/ObjC but not for C++ [enabled by default]
g++ -pthread -shared -L/home/lynx/anaconda2/envs/cython-example/lib -Wl,-rpath=/home/lynx/anaconda2/envs/cython-example/lib,--no-as-needed build/temp.linux-x86_64-2.7/src/taylor.o build/temp.linux-x86_64-2.7/src/taylor_series.o -L/home/lynx/anaconda2/envs/cython-example/lib -lpython2.7 -o /home/lynx/src/c++/numcom/taylor_series/cython/taylor.so
(cython-example) [lynx@lille-login2 cython]$ ls
build  setup.py  src  taylor.so
```

The library `taylor.so` has been built. In addtion the build process has genenrated
a `src/taylor.cpp` and a subdirectory `build`:

```shell
.
├── build
│   └── temp.linux-x86_64-2.7
│       └── src
│           ├── taylor.o
│           └── taylor_series.o
├── setup.py
├── src
│   ├── taylor.cpp
│   ├── taylor.pyx
│   ├── taylor_series.cpp
│   └── taylor_series.h
└── taylor.so

4 directories, 8 files
```

We can now test our library in python:

```python
Python 2.7.13 |Continuum Analytics, Inc.| (default, Dec 20 2016, 23:09:15) 
[GCC 4.4.7 20120313 (Red Hat 4.4.7-1)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
Anaconda is brought to you by Continuum Analytics.
Please check out: http://continuum.io/thanks and https://anaconda.org
>>> import taylor.so
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ImportError: No module named so
>>> import taylor
>>> taylor.sin(3.141592653/3,15)
0.8660254036861398
>>> 
```

### PyBind11
Make a subdirectory pybind11 with an additional src subdirectory:

```shell
mkdir -p pybind11/src
cd pybind11/src
```
In src subdirectory we will have three files:
```shell
(pybind11-example) [lynx@lsrc]$ ls
py11taylor.cpp  taylor_series.cpp  taylor_series.h
```
The contents of taylor_series.cpp and taylor_series.h will we recognize from the SWIG-example, with a few differences:
```cpp
// taylor_series.h
#ifndef TAYLOR_SERIES_H_
#define TAYLOR_SERIES_H_

unsigned long long factorial( int n);
double ts_sin(double& x, int N);
double ts_cos(double& x,int N);

#endif // TAYLOR_SERIES_H_
```

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
Create these files with your favorite editor, and leave the subdirectory.
In the directory above we make CMakeLists.txt with following contents:
```cmake
cmake_minimum_required(VERSION 2.8 FATAL_ERROR)
project(taylor)
find_package(pybind11 REQUIRED)
include_directories(src)
set(SOURCE_FILES src/taylor_series.cpp src/py11taylor.cpp)
pybind11_add_module(taylor ${SOURCE_FILES})
```

```shell
(pybind11-example) [lynx@src]$ ls
py11taylor.cpp  taylor_series.cpp  taylor_series.h
(pybind11-example) [lynx@src]$ cd ..
(pybind11-example) [lynx@pybind11]$ vi CMakeLists.txt
(pybind11-example) [lynx@pybind11]$ ls
CMakeLists.txt  src
(pybind11-example) [lynx@pybind11]$ mkdir build
(pybind11-example) [lynx@pybind11]$ cd build
(pybind11-example) [lynx@build]$ cmake ..
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
-- Build files have been written to: /home/lynx/tmp/pybind11/build
(pybind11-example) [lynx@build]$ make
Scanning dependencies of target taylor
[ 50%] Building CXX object CMakeFiles/taylor.dir/src/taylor_series.cpp.o
[100%] Building CXX object CMakeFiles/taylor.dir/src/py11taylor.cpp.o
Linking CXX shared module taylor.so
[100%] Built target taylor
(pybind11-example) [lynx@build]$ python
Python 2.7.13 |Continuum Analytics, Inc.| (default, Dec 20 2016, 23:09:15)
[GCC 4.4.7 20120313 (Red Hat 4.4.7-1)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
Anaconda is brought to you by Continuum Analytics.
Please check out: http://continuum.io/thanks and https://anaconda.org
>>> import taylor
>>> taylor.sin(3.141592653/3,14)
0.8660254036861398
>>>


```shell
source deactivate
```




