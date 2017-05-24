---
layout: episode
title: Building mixed-language projects with CMake
teaching: 30
exercises: 0
questions:
  - Write me.
objectives:
  - Write me.
keypoints:
  - Write me.
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


```
Here we introduce an example C++ project which we hook up to PyBind11 in the
next section.

We need a hands-on early in the workshop to keep people motivated and engaged.
Try to have an exercise not more than 20 minutes into the workshop.
