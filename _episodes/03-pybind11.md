---
layout: episode
title: Hands-on example using pybind11
teaching: 15
exercises: 15
questions:
  - What tool do you recommend to interface Python and C++?
objectives:
  - Unit testing and prototyping of C++ code.
#keypoints:
#  - Write me.
---

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
$ mkdir build
$ cd build
$ cmake ..

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

$ make

[ 33%] Compiling Cython CXX source for taylor...
Scanning dependencies of target taylor
[ 66%] Building CXX object src/CMakeFiles/taylor.dir/taylor.cxx.o
[100%] Building CXX object src/CMakeFiles/taylor.dir/taylor_series.cpp.o
Linking CXX shared module ../lib/taylor.so
[100%] Built target taylor

$ cd lib
$ python

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
$ python setup.py build_ext -i

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

$ python
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
$ mkdir build
$ cd build
$ cmake ..

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

$ make

Scanning dependencies of target taylor
[ 50%] Building CXX object src/CMakeFiles/taylor.dir/py11taylor.cpp.o
[100%] Building CXX object src/CMakeFiles/taylor.dir/taylor_series.cpp.o
Linking CXX shared module ../lib/taylor.so
[100%] Built target taylor

$ cd lib
$ python

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
$ git clone https://github.com/blindij/python-ctools-demo.git

Cloning into 'python-ctools-demo'...
remote: Counting objects: 81, done.
remote: Compressing objects: 100% (67/67), done.
remote: Total 81 (delta 18), reused 73 (delta 10), pack-reused 0
Unpacking objects: 100% (81/81), done.

$ cd python-ctools-demo/
$ git submodule init

Submodule 'phoneticA/pybind11' (https://github.com/pybind/pybind11.git) registered for path 'phoneticA/pybind11'
Submodule 'pybind11-demo/pybind11' (https://github.com/pybind/pybind11.git) registered for path 'pybind11-demo/pybind11'

$ git submodule update

Cloning into 'pybind11-demo/pybind11'...
remote: Counting objects: 8182, done.
remote: Compressing objects: 100% (17/17), done.
remote: Total 8182 (delta 4), reused 11 (delta 3), pack-reused 8161
Receiving objects: 100% (8182/8182), 2.85 MiB | 1.12 MiB/s, done.
Resolving deltas: 100% (5444/5444), done.
Submodule path 'pybind11-demo/pybind11': checked out '13d8cd2cc7566de34d724f428ea7a6b6448d6a0c'
```

## Pybind11  - binding to C++11

Assume we are starting a project on  Phonetic Algorithms. We have received
source code for the Soundex algorithm which is one Phonetic Algorithm.
(The code is from  [Jeff Langr's](http://langrsoft.com/about/) book
[Modern C++ Programming with Test-Driven Development](http://pragprog.com/book/lotdd/modern-c-programming-with-test-driven-development))

Our ambition is to implement a Phonetic Algorithm class where Soundex is one
several encoding algorithm. Here is class diagram of what we want to implement:
![Class diagram for phoneticA]({{ site.baseurl }}/img/uml-phoneticA.png "Class diagram for phoneticA. Licenses CC BY 3.0"){:class="img-responsive"}

Other Phonetic Algorithms are [Match Rating Approach](https://en.wikipedia.org/wiki/Match_rating_approach),
[Daitch-Mokotoff Soundex](https://en.wikipedia.org/wiki/Daitch–Mokotoff_Soundex), or [New Your State Identification and Intelligence System (NYIIS)](https://en.wikipedia.org/wiki/New_York_State_Identification_and_Intelligence_System)

### The setup
Change directory to the `phoneticA` subdirectory:

```bash
$ pwd
/Users/lynx/git-repos/python-ctools-demo/pybind11-demo
$ cd ../phoneticA/
```

You should have filestructure which is similar, with a lot of files under `pybind11`
If `pybind11` is empty, do `git submodule init; git submodule update`.


```bash
.
|-- CMakeLists.txt
|-- pybind11
...
`-- src
    |-- CMakeLists.txt
    |-- PhoneticAlgorithm.h
    |-- Py11PhoneAlg.cpp
    `-- Soundex.h
```

### The C++ source code
Here is source code for the virtual class  PhoneticAlgorithm. The class have
one pure virtual function. In C++ this means all classes which
inherit the class PhoneticAlgorithm need to implement this function.

```cpp
// file: PhoneticAlgorithm.h
#ifndef PHONETICALGORITHM_H
#define PHONETICALGORITHM_H

class PhoneticAlgorithm{
 public:
  virtual ~PhoneticAlgorithm(){ }
  virtual std::string encode(const std::string& word) const = 0;  // pure virtual function
};

#endif //PHONETICALGORITHM_H
```

Here is the implementation of the Soundex algorithm.

```cpp
// file: Soundex.h
#ifndef SOUNDEX_H
#define SOUNDEX_H
#include <string>
#include <unordered_map>

#include "PhoneticAlgorithm.h"

// Soundex inherit PhoneticAlgorithm

class Soundex : public PhoneticAlgorithm {
    static const size_t MaxCodeLength{4};
public:
    std::string encode(const std::string& word) const {
        return zeroPad(upperFront(head(word))+tail(encodedDigits(word)));
    }

    std::string encodedDigit(char letter) const {
        const std::unordered_map<char,std::string> encodings{
                {'b',"1"},{'f',"1"},{'p',"1"},{'v',"1"},
                {'c',"2"},{'g',"2"},{'j',"2"},{'k',"2"},{'q',"2"},{'s',"2"},{'x',"2"},{'z',"2"},
                {'d',"3"},{'t',"3"},
                {'l',"4"},
                {'m',"5"},{'n',"5"},
                {'r',"6"}

        };
        auto it = encodings.find(lower(letter));
        return it == encodings.end() ? NotADigit: it->second;
    }

private:
    const std::string NotADigit{"*"};

    char lower(char c) const {
        return std::tolower(static_cast<unsigned char>(c));
    }

    std::string head(const std::string& word) const {
        return word.substr(0,1);
    }

    std::string tail(const std::string& word) const {
        return word.substr(1);
    }

    std::string encodedDigits(const std::string& word) const{
        std::string encoding;
        encodeHead(encoding, word);
        encodeTail(encoding,word);
        return encoding;
    }

    void encodeHead(std::string& encoding, const std::string& word) const {
        encoding += encodedDigit(word.front());
    }

    void encodeTail(std::string& encoding, const std::string& word) const{
        for (auto i=1u; i < word.length();i++)  {
            if (!isComplete(encoding))
                encodeLetter(encoding,word[i],word[i-1]);
        }
    }

    void encodeLetter(std::string& encoding, char letter, char lastLetter) const {
        auto digit = encodedDigit(letter);
        if ( digit != NotADigit && ( digit != lastDigit(encoding) || isVowel(lastLetter)))
            encoding += digit;
    }

    bool isComplete(const std::string& encoding) const{
        return encoding.length() == MaxCodeLength;
    }

    std::string zeroPad(const std::string& word) const {
        auto zerosNeeded = MaxCodeLength - word.length();
        return word + std::string(zerosNeeded,'0');
    }

    std::string lastDigit(const std::string& encoding) const {
        if (encoding.empty()) return NotADigit;
        return std::string(1,encoding.back());

    }

    std::string upperFront(const std::string& string) const {
        return std::string(1, std::toupper(static_cast<unsigned char>(string.front())));
    }

    bool isVowel(char letter) const {
        return std::string("aeiouy").find(lower(letter)) != std::string::npos;
    }
};

#endif //SOUNDEX_H
```

The [Soundex algorithm](https://en.wikipedia.org/wiki/Soundex), according to Wikipedia, maps a name or word to its' first letter
followed by three numerical digits. Outlined the algorithm goes like:
 1. Retain the first letter of the name and drop all other occurrences of a,e,i,o,u,y,h,w
 2. Replace consonants with digits as follows (after the first letter)
    * b,f,p,v -> 1
    * c,g,j,k,q,s,x,z, -> 2
    * d,t -> 3
    * m,n -> 5
    * r -> 6

 3. If two or more letters with the same number are adjacent in the original name
 (before step 1), only retain the first letter; also two letters with the same
 number separated by 'h' or 'w' are coded as a single number, whereas such letters
 separated by a vowel are coded twice. This rule also applies to the first letter.
 4. If you have too few letters in your word that you can't assign three numbers,
 append with zeros until there are three numbers. If you have more than 3 letters,
 just retain the first 3 numbers.

### The interface file
The interface for Python needs to be defined. This is done in `Py11PhoneAlg.cpp`.

```cpp
// file: Py11PhoneAlg.cpp
#include <pybind11/pybind11.h>
#include "PhoneticAlgorithm.h"
#include "Soundex.h"

namespace py = pybind11;

class PyPhoneticAlg : public PhoneticAlgorithm {
public:
  // Inherit the constructors
  using PhoneticAlgorithm::PhoneticAlgorithm;
  std::string encode(const std::string& word) const override {
    PYBIND11_OVERLOAD_PURE (
			    std::string,      // Return type
			    PhoneticAlgorithm, // Parent class
			    encode,            // Name of function in C++
			    word               // Argument(s)
			    );
  }
};

std::string call_encode(PhoneticAlgorithm& phxalg) {
  return phxalg.encode("Allison");
}

PYBIND11_MODULE(phoneticA, mod) {
  mod.doc() = "pybind11 phonetic plugin";


  py::class_<PhoneticAlgorithm, PyPhoneticAlg> phalgo(mod,"phalgo");
  phalgo
    .def(py::init<>())
    .def("encode", &PhoneticAlgorithm::encode);


  py::class_<Soundex>(mod,"sndx",phalgo)
    .def(py::init<>())
    .def("encode", &Soundex::encode);


  mod.def("call_encode", &call_encode);

}
```

### Building the library
There is three CMakeLists.txt files in the directory. One is part of the `pybind11`
project. The two others are made for the this project, phoneticA. Here is the top
level CMake file

```cmake
cmake_minimum_required(VERSION 2.8 FATAL_ERROR)
project(phoneticA)

SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY
   ${CMAKE_BINARY_DIR}/lib
   )

add_subdirectory(pybind11)
add_subdirectory(src)
```


This is the CMake file in the `src` subdirectory:

```cmake
include_directories(${CMAKE_CURRENT_SOURCE_DIR})
set(SOURCE_FILES Py11PhoneAlg.cpp)
pybind11_add_module(phoneticA ${SOURCE_FILES})
```

To build this project and load it in Python we do the following at the top level directory:

```shell
$ mkdir build
$ cd build
$ cmake ..
$ make
$ cd lib
$ python
>>> import phoneticA
>>> d = phoneticA.sndx()
>>> d.encode("Robertsen")
```

Here is output from building on a Linux system:

```shell
[lynx@lille-login2 python-ctools-demo]$ cd phoneticA/
[lynx@lille-login2 phoneticA]$ mkdir build
[lynx@lille-login2 phoneticA]$ cd build
[lynx@lille-login2 build]$ cmake ..
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
-- Found PythonInterp: /usr/bin/python (found version "2.7.5")
-- Found PythonLibs: /usr/lib64/libpython2.7.so
-- pybind11 v2.2.dev0
-- Performing Test HAS_CPP14_FLAG
-- Performing Test HAS_CPP14_FLAG - Failed
-- Performing Test HAS_CPP11_FLAG
-- Performing Test HAS_CPP11_FLAG - Success
-- Performing Test HAS_FLTO
-- Performing Test HAS_FLTO - Success
-- LTO enabled
-- Configuring done
-- Generating done
-- Build files have been written to: /home/lynx/tmp/python-ctools-demo/phoneticA/build
[lynx@lille-login2 build]$ make
Scanning dependencies of target phoneticA
[100%] Building CXX object src/CMakeFiles/phoneticA.dir/Py11PhoneAlg.cpp.o
Linking CXX shared module ../lib/phoneticA.so
[100%] Built target phoneticA
[lynx@lille-login2 build]$ cd lib
[lynx@lille-login2 lib]$ python
Python 2.7.5 (default, Nov  6 2016, 00:28:07)
[GCC 4.8.5 20150623 (Red Hat 4.8.5-11)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import phoneticA
>>> d = phoneticA.sndx()
>>> d.encode("Robertsen")
u'R163'
>>>
```

### Prototyping a new class - Match Rating Approach(MRA)

[The Match Rating Approach](https://en.wikipedia.org/wiki/Match_rating_approach) goes like:
 1. Delete all vowels unless the vowel begins the word
 2. Remove the second constant of any double consontants present
 3. Reduce codex to 6 letters by joining the first and last 3 letters only

We start implementing a class MRA in Python which inherit PhoneticAlgorithm,
and encode() do the first rule of MRA.

```python
>>>
>>> import  phoneticA
>>> d = phoneticA.sndx()
>>> import  re
>>> vwls=re.compile(r'[aeiouy]',re.IGNORECASE)
>>> re.sub(vwls,"","Robertson")
'Rbrtsn'
>>> class MRA(phoneticA.phalgo):
...     def encode(this, word):
...             return word[0]+re.sub(vwls,"",word[1:])
...
>>> x = MRA()
>>> x.encode("Robertson")
'Rbrtsn'
>>> x.encode("Anderson")
'Andrsn'
>>>
>>> phoneticA.call_encode(x)
u'Allsn'
>>> phoneticA.call_encode(d)
u'A425'
```

The function calls to `call_encode()` above shows polymorphism in play.
The polyphorism is "imported" from the C++ side, but we make use of it on the
python side.

### Extending the classes dynamically

In Python you can add new attributes to a class dynamically. As the Phonetic/Algorithm
and Soundex class are implemented, they can not get new attributes dynamically.

Though, we can add this feature to the PhoneticAlgorithm. The subclasses will inherit it.

The Py11PhoneAlg.cpp is changed to:

```cpp
// file: Py11PhoneAlg.cpp
#include <pybind11/pybind11.h>
#include "PhoneticAlgorithm.h"
#include "Soundex.h"

namespace py = pybind11;

class PyPhoneticAlg : public PhoneticAlgorithm {
public:
  // Inherit the constructors
  using PhoneticAlgorithm::PhoneticAlgorithm;
  std::string encode(const std::string& word) const override {
    PYBIND11_OVERLOAD_PURE (
			    std::string,      // Return type
			    PhoneticAlgorithm, // Parent class
			    encode,            // Name of function in C++
			    word               // Argument(s)
			    );
  }
};

std::string call_encode(PhoneticAlgorithm& phxalg) {
  return phxalg.encode("Allison");
}

PYBIND11_MODULE(phoneticA, mod) {
  mod.doc() = "pybind11 phonetic plugin";

  py::class_<PhoneticAlgorithm, PyPhoneticAlg>(mod,"phalgo",py::dynamic_attr())
    .def(py::init<>())
    .def("encode", &PhoneticAlgorithm::encode);

  py::class_<Soundex,PhoneticAlgorithm>(mod,"sndx")
    .def(py::init<>())
    .def("encode", &Soundex::encode);

  mod.def("call_encode", &call_encode);

}
```

We add the `py::dynamic_attr` tag to the  `py::class_` constructor. In addition
we state the inheritance on the "C++"-side of the Soundex() `py::class_` constructor.

We build the module once more, assuming we are done in the `src`-directory:

```bash
$ cd ..
$ cd build
$ make
$ cd lib
$ python
```

Import the module into Python again, and let us check out how the classes have changed.

```python
>>> import phoneticA
>>> p = phoneticA.sndx()
>>> p.__dict__
{}
>>> p.original="Longhorn"
>>> p.__dict__
{'original': 'Longhorn'}
>>> class reversesoundex(phoneticA.phalgo):
...     def encode(this, word):
...             t = phoneticA.sndx()
...             tmp = t.encode(word)
...             return tmp[1:]+tmp[0]
...
>>> r = reversesoundex()
>>> r.encode('Robertson')
u'163R'
>>> r.__dict__
{}
>>> r.orginal='Robertson'
>>> r.__dict__
{'orginal': 'Robertson'}
>>>
```
In this way we can extend the individual classes with attributes or instance
methods whether or not the classes are implemented in C++ or Python.

LocalWords:  Soundex Pybind11
