---
layout: episode
title: Building mixed-language projects with CMake
teaching: 30
exercises: 0
questions:
  - How to to deal with all this complexity?
objectives:
  - CMake reduces the complexity in the building process
keypoints:
  - With reduce complexity, and a swifter turn around in the building process
    combining a scripting language and source code for a compiled
    language become much efficient.
---

## Introducing CMake
## About [CMake](https://cmake.org)

- Cross-platform
- Open-source
- [Actively developed](https://github.com/Kitware/CMake)
- Manages the build process in a compiler-independent manner
- Designed to be used in conjunction with the native build environment
- Written in C++ (does not matter but if you want to build CMake yourself, C++ is all you need)
- CMake is a generator - this means it does not compile the code
- Not a replacement for Makefiles, rather a replacement for Autotools

---

## Why CMake?

Separation of source and build path:

- Out-of-source compilation (possibility to compile several builds with the same source)

Portability:

- Really cross-platform (Linux, Mac, Windows, AIX, iOS, Android)
- CMake defines portable variables about the system
- Cross-platform system- and library-discovery

Language support:

- Excellent support for Fortran, C, C++, and Java, as well as mixed-language projects
- CMake understands Fortran 90 dependencies very well; no need to program a dependency scanner
- Excellent support for multi-component and multi-library projects

Supports modular code development:

- Makes it possible and relatively easy to download, configure, build, install, and link external modules

Provides tools:

- Generates user interface (command-line or text-UI or GUI)
- Full-fledged testing and packaging framework with CTest and CPack
- CTest uses a Makefile (possible to run sequential tests concurrently)

Popular:

- CMake is used by many prominent projects:
  MySQL, Boost, VTK, Blender, KDE, LyX, Mendeley, MikTeX, Compiz, Google Test, ParaView, Second Life, Avogadro, and many more ...

## Hello-world example

We start with our now familiar C++ program:

```cpp
#include <iostream>

int main()
{
    std::cout << "Hello World!" << std::endl;
}
```

Create a fresh directory and save the C++ code to a file called
`hello.cpp`.

We wish to compile this code to `hello.x`.

For this we create a file called `CMakeLists.txt` which contains:

```cmake
# stop if cmake version is below 2.8
cmake_minimum_required(VERSION 2.8 FATAL_ERROR)

# project name
project(hello)

# we enable C++ support
enable_language(CXX)

# we define the executable and its dependencies
add_executable(hello.x hello.cpp)
```

Your directory should look like this:

```shell
$ ls

CMakeLists.txt	hello.cpp
```

Now we create a build directory (out of source compilation), change to it,
and configure the project:

```shell
$ mkdir build
$ cd build/
$ cmake ..

-- The C compiler identification is GNU 6.2.1
-- The CXX compiler identification is GNU 6.2.1
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
-- Configuring done
-- Generating done
-- Build files have been written to: /home/user/example/build
```

Now we are ready to compile the code:

```shell
$ make

Scanning dependencies of target hello.x
[ 50%] Building CXX object CMakeFiles/hello.x.dir/hello.cpp.o
[100%] Linking CXX executable hello.x
[100%] Built target hello.x
```

Done. In the following we will learn what happened here behind the scenes.

Have you used `git init`? Good! You probably want to ignore the `build/` directory.

---

## Name and location of the build directory

We have configured and compiled the code like this:

```shell
$ mkdir build
$ cd build/
$ cmake ..
$ make
```

This is very similar to the steps taken when building taylor library with
Pybind11. Here is the CMakeLists.txt from that example:
```cmake
cmake_minimum_required(VERSION 2.8 FATAL_ERROR)
project(taylor)
find_package(pybind11 REQUIRED)
include_directories(src)
set(SOURCE_FILES src/taylor_series.cpp src/py11taylor.cpp)
pybind11_add_module(taylor ${SOURCE_FILES})
```
Here we require that Pybind11 exists if not the generation will stop. We add
'src' as a subdirectory to include ( equals -I src when passing arguments to
compiler). The command set enables a environmental variable, here call SOURCE_FILES.
The variable is used in the pybind11_add_module() which is a CMake module
provided by the Pybind11 installation. pybind11_add_module() generates steps
necessary for building the python extension taylor.so.

## CMake and SWIG

SWIG can also use CMake as generator for the building process. This time we
make use of the same source code as in the Pybind11 example, we pass the
arguments by reference, as we would expect of C++ code. This requires some
changes to our interface file:
```swig
//
// file: taylor.i
// SWIG - interface file
//
%module taylor
%{
  // include C++ header
#include "taylor_series.h"
  %}

%include "typemaps.i"
%apply double *INPUT {double& x }
%include "taylor_series.h"

```
We put the interface file in a src subdirectory together with the source files:
```shell
[lynx@~]$mkdir -p swigcmake/src
[lynx@~]$cd swigcmake/src
```
After making the files the src subdirectory will contain:
```shell
[lynx@src]$ ls
taylor.i  taylor_series.cpp  taylor_series.h
[lynx@src]$ cd ..
[lynx@swigcmake]$
```
Move up one directory. Here we will make a CMakeLists.txt. It will contain:
```cmake
cmake_minimum_required(VERSION 2.8 FATAL_ERROR)

SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY
   ${CMAKE_BINARY_DIR}/lib
   )

FIND_PACKAGE(SWIG REQUIRED)
INCLUDE(${SWIG_USE_FILE})

FIND_PACKAGE(PythonLibs)

ADD_SUBDIRECTORY(src)
```
Here we state that the library we will make will be output to a lib subdirectory.
The SWIG library is necessary. The SWIG package will set the ${SWIG_USE_FILE}
environment variable. The variable points to CMake file which will be loaded and
executed by the INCLUDE statement. The find_package(PythonLIbs) finds python.h and
the python libraries. At the end we add the 'src'-subdirectory. CMake will search
this directory for an additional CMakeLists.txt.

In the 'src'-subdirectory we make a CMakeLists.txt containing:
```cmake
INCLUDE_DIRECTORIES(${PYTHON_INCLUDE_PATH})
INCLUDE_DIRECTORIES(${CMAKE_CURRENT_SOURCE_DIR})

SET(CMAKE_SWIG_FLAGS "")

SET_SOURCE_FILES_PROPERTIES(taylor.i PROPERTIES CPLUSPLUS ON)
SET_SOURCE_FILES_PROPERTIES(taylor.i PROPERTIES SWIG_FLAGS "-includeall")

SWIG_ADD_MODULE(taylor python taylor.i taylor_series.cpp)
SWIG_LINK_LIBRARIES(taylor ${PYTHON_LIBRARIES})
```
Our 'src'-subdirectory contains:
```shell
(swig-example) [lynx@src]$ ls
CMakeLists.txt  CMakeLists.txt~  taylor.i  taylor_series.cpp  taylor_series.h
(swig-example) [lynx@src]$ cd ..
(swig-example) [lynx@swigexample]$ mdkir build;cd build
(swig-example) [lynx@build]$ cmake ..
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
-- Found SWIG: /home/lynx/anaconda2/envs/swig-example/bin/swig (found version "3.0.10")
-- Found PythonLibs: /usr/lib64/libpython2.7.so (found version "2.7.5")
-- Configuring done
-- Generating done
-- Build files have been written to: /home/lynx/src/c++/numcom/taylor_series/swig/build
(swig-example) [lynx@build]$ make
[ 33%] Swig source
Scanning dependencies of target _taylor
[ 66%] Building CXX object src/CMakeFiles/_taylor.dir/taylorPYTHON_wrap.cxx.o
[100%] Building CXX object src/CMakeFiles/_taylor.dir/taylor_series.cpp.o
Linking CXX shared module ../lib/_taylor.so
[100%] Built target _taylor
(swig-example) [lynx@build]$ cd lib
(swig-example) [lynx@lib]$ python
Python 2.7.13 |Continuum Analytics, Inc.| (default, Dec 20 2016, 23:09:15)
[GCC 4.4.7 20120313 (Red Hat 4.4.7-1)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
Anaconda is brought to you by Continuum Analytics.
Please check out: http://continuum.io/thanks and https://anaconda.org
>>> import _taylor as taylor
>>> taylor.sin(3.141592356/3,15)
0.8660253541861354
>>>
```
