---
layout: episode
title: Overview over different approaches
teaching: 10
exercises: 0
questions:
  - Write me.
objectives:
  - Write me.
keypoints:
  - Write me.
---

## Section
There exist several technologies which make this possible:

* SWIG
* F2PY
* Boost
* Pybind11
* Cython

Simplified Wrapper and Interface Generator (SWIG) is a tool that simplies the two step process of making a wrapper and generating a interface which makes the wrapper callable from the interpreter. According to the SWIG documentation, "SWIG was orignally designed to make it extremely easy for scientist and engineers to build extensible scientific software without having a degree in software engineering". So SWIG should really be the only thing we need, right? Could be, but before giving a motivation for the other tools, it is worth mentioning that SWIG support a range of interpreting languages, not only Python.

F2PY is a tool for interfacing Fortan and Python. According to "Python Scripting for Computational Science" transfering Numpy arrays between Python and compiled Fortran code is easier with F2PY than SWIG.

Boost is a huge C++-library which works with almost any C++-compiler. The Python interface tool was added to the Boost library around 2002 by David Abrahams. Hence, if you a have special C++-compiler, Boost.Python could be your most suitable tool for integrating Python and C++.

"Pybind11 is a lightweight-header only library that exposes C++-types in Python and vice versa, mainly to create Python bindings of existing C++ code.", according to the Pybind11 web page, https://pybind11.readthedocs.io/en/stable/intro.html. The point it is lightweight and targeting the combination of Python and C++11compliant compilers. Consequently, the interface code becomes more straight forward.

Cython:"All of this makes Cython the ideal language for wrapping external C libraries, embedding CPython into existing applications, and for fast C modules that speed up the execution of Python code.", see webpage http://cython.org


##SWIG
Our source code contains three functions, Taylor series of sin(), cos() and a helper function factorial(). We will make these functions available in our Python interpreter with the use of SWIG

``` Simplified Wrapper and Interface Generator,http://www.swig.org/ (SWIG). It is a software tool for making programs/applications written C/C++ accessible from a high-level programming language. Python is on of these high-level programming languages, but there is a range of others(C#,Common Lisp, Go,R, Lua et cetera).
```
Start out with at empty subdirectory, your C++ compiler and the Anaconda2 enviroment available in your path:
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

```
Create a SWIG enabled environment in conda. Here we also add scipy to have available for possible comparisons with the methods we have implemented.


``` shell
[lynx@swig]$echo ${PATH}
/home/lynx/anaconda2/bin:/home/lynx/local/bin:/usr/lib64/qt-3.3/bin:/usr/local/bin:/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/sbin:/opt/ganglia/bin:/opt/ganglia/sbin:/usr/java/latest/bin:/opt/pdsh/bin:/opt/rocks/bin:/opt/rocks/sbin:/home/lynx/bin

[lynx@swig]$conda create --name swig-example                         
Fetching package metadata .........
Solving package specifications: 
Package plan for installation in environment /home/lynx/anaconda2/envs/swig-example:

Proceed ([y]/n)? y

#
# To activate this environment, use:
# > source activate swig-example
#
# To deactivate this environment, use:
# > source deactivate swig-example
#


[lynx@swig]$source activate swig-example
```

Note how you get '(swig-example)' at the start of your prompt when you have activated this environment:

```shell
(swig-example) [lynx@swig]$ conda install scipy swig
Fetching package metadata .........
Solving package specifications: .

Package plan for installation in environment /home/lynx/anaconda2/envs/swig-example:

The following NEW packages will be INSTALLED:

    libgfortran: 3.0.0-1           
    mkl:         2017.0.1-0        
    numpy:       1.12.1-py27_0     
    openssl:     1.0.2k-2          
    pcre:        8.39-1            
    pip:         9.0.1-py27_1      
    python:      2.7.13-0          
    readline:    6.2-2             
    scipy:       0.19.0-np112py27_0
    setuptools:  27.2.0-py27_0     
    sqlite:      3.13.0-0          
    swig:        3.0.10-0          
    tk:          8.5.18-0          
    wheel:       0.29.0-py27_0     
    zlib:        1.2.8-3           

Proceed ([y]/n)? y
libgfortran-3. 100% |##########################################| Time: 0:00:00   6.36 MB/s
mkl-2017.0.1-0 100% |##########################################| Time: 0:00:02  66.96 MB/s
openssl-1.0.2k 100% |##########################################| Time: 0:00:00  70.17 MB/s
readline-6.2-2 100% |##########################################| Time: 0:00:00  62.06 MB/s
sqlite-3.13.0- 100% |##########################################| Time: 0:00:00  72.32 MB/s
tk-8.5.18-0.ta 100% |##########################################| Time: 0:00:00  70.14 MB/s
zlib-1.2.8-3.t 100% |##########################################| Time: 0:00:00  45.90 MB/s
pcre-8.39-1.ta 100% |##########################################| Time: 0:00:00  62.08 MB/s
python-2.7.13- 100% |##########################################| Time: 0:00:00  72.23 MB/s
numpy-1.12.1-p 100% |##########################################| Time: 0:00:00  74.48 MB/s
setuptools-27. 100% |##########################################| Time: 0:00:00  59.51 MB/s
swig-3.0.10-0. 100% |##########################################| Time: 0:00:00  69.37 MB/s
wheel-0.29.0-p 100% |##########################################| Time: 0:00:00  40.78 MB/s
pip-9.0.1-py27 100% |##########################################| Time: 0:00:00  67.90 MB/s
scipy-0.19.0-n 100% |##########################################| Time: 0:00:00  71.99 MB/s

```
Now we create a src subdirectory with the source files taylor_series.h and taylor_series.h

Here is the contents of the taylor_series.h:
```C++
#ifndef TAYLOR_SERIES_H_
#define TAYLOR_SERIES_H_

extern unsigned long long factorial( int n);
extern double sin(double x, int N);
extern double cos(double x,int N);

#endif // TAYLOR_SERIES_H_

```

The contents of taylor_series.cpp:

```C++
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
Now we need a interface file which describes how these functions can be called. In SWIG nomenclature this is a i-file. Here we define the module name and state which functions that needs wrappers.



### By reference

Here is our C++-source code of the Taylor Series, the taylor_series.cpp:
``` C++
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

```C++
#ifndef TAYLOR_SERIES_H_
#define TAYLOR_SERIES_H_

extern unsigned long long factorial( int n);
extern double ts_sin(double& x, int N);
extern double ts_cos(double& x,int N);

#endif // TAYLOR_SERIES_H_

```

To make these functions available for the Python interpreter with the use of SWIG, we need to write a interface file, a %.i file. In our example called tss.i:
```C++
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

## Installation of Anaconda - the Conda environment
We will create three different work environments in Anaconda. Here it is assumed that you have anaconda2 installed. Below is the necessary steps to install Anconda2 in your home area as ~/anaconda2

```shell
curl -OL https://repo.continuum.io/archive/Anaconda2-4.3.1-Linux-x86_64.sh
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  462M  100  462M    0     0  91.8M      0  0:00:05  0:00:05 --:--:--  102M
md5sum Anaconda2-4.3.1-Linux-x86_64.sh 
51336ab38e15ce607b55539c60be2c29  Anaconda2-4.3.1-Linux-x86_64.sh
sh ./Anaconda2-4.3.1-Linux-x86_64.sh 
...
[ yes to license ]
...
Anaconda2 will now be installed into this location:
/home/nlnj/anaconda2

  - Press ENTER to confirm the location
  - Press CTRL-C to abort the installation
  - Or specify a different location below

[/home/lynx/anaconda2] >>> 
PREFIX=/home/lynx/anaconda2
installing: python-2.7.13-0 ...
installing: _license-1.1-py27_1 ...
installing: alabaster-0.7.9-py27
...
Python 2.7.13 :: Continuum Analytics, Inc.
creating default environment...
installation finished.
Do you wish the installer to prepend the Anaconda2 install location
to PATH in your /home/lynx/.bashrc ? [yes|no]
[no] >>> yes

Prepending PATH=/home/lynx/anaconda2/bin to PATH in /home/lynx/.bashrc
A backup will be made to: /home/lynx/.bashrc-anaconda2.bak


For this change to become active, you have to open a new terminal.

Thank you for installing Anaconda2!

Share your notebooks and packages on Anaconda Cloud!
Sign up for free: https://anaconda.org
```

We then proceed to create the three work environments need for the examples, assuming that the Anaconda2 binary subdirectory is first in your PATH.




##Boost
```shell
[lynx@login-0-0 Downloads]$ conda create --name boost-example
Fetching package metadata .........
Solving package specifications: 
Package plan for installation in environment /home/lynx/anaconda2/envs/boost-example:

Proceed ([y]/n)? y

#
# To activate this environment, use:
# > source activate boost-example
#
# To deactivate this environment, use:
# > source deactivate boost-example
#

[lynx@login-0-0 Downloads]$ source activate
(root) [lynx@login-0-0 Downloads]$ source deactivate
[lynx@login-0-0 Downloads]$ source activate boost-example
(boost-example) [lynx@login-0-0 Downloads]$ conda install scipy boost
Fetching package metadata .........
Solving package specifications: .

Package plan for installation in environment /home/lynx/anaconda2/envs/boost-example:

The following NEW packages will be INSTALLED:

    boost:       1.61.0-py27_0     
    icu:         54.1-0            
    libgfortran: 3.0.0-1           
    mkl:         2017.0.1-0        
    numpy:       1.12.1-py27_0     
    openssl:     1.0.2k-2          
    pip:         9.0.1-py27_1      
    python:      2.7.13-0          
    readline:    6.2-2             
    scipy:       0.19.0-np112py27_0
    setuptools:  27.2.0-py27_0     
    sqlite:      3.13.0-0          
    tk:          8.5.18-0          
    wheel:       0.29.0-py27_0     
    zlib:        1.2.8-3           

Proceed ([y]/n)? y

icu-54.1-0.tar 100% |##########################################| Time: 0:00:00  13.09 MB/s
boost-1.61.0-p 100% |##########################################| Time: 0:00:01  16.30 MB/s

source deactivate
```


##F2PY

##Cython

##PyBind11

```shell
[lynx@login-0-0 Downloads]$ conda create --name pybind11-example
Fetching package metadata .........
Solving package specifications: 
Package plan for installation in environment /home/lynx/anaconda2/envs/pybind11-example:

Proceed ([y]/n)? y

#
# To activate this environment, use:
# > source activate pybind11-example
#
# To deactivate this environment, use:
# > source deactivate pybind11-example
#

[lynx@login-0-0 Downloads]$ source activate pybind11-example
(pybind11-example) [lynx@login-0-0 Downloads]$ conda install scipy         
Fetching package metadata .........
Solving package specifications: .

Package plan for installation in environment /home/lynx/anaconda2/envs/pybind11-example:

The following NEW packages will be INSTALLED:

    libgfortran: 3.0.0-1           
    mkl:         2017.0.1-0        
    numpy:       1.12.1-py27_0     
    openssl:     1.0.2k-2          
    pip:         9.0.1-py27_1      
    python:      2.7.13-0          
    readline:    6.2-2             
    scipy:       0.19.0-np112py27_0
    setuptools:  27.2.0-py27_0     
    sqlite:      3.13.0-0          
    tk:          8.5.18-0          
    wheel:       0.29.0-py27_0     
    zlib:        1.2.8-3           

Proceed ([y]/n)? 

(pybind11-example) [lynx@login-0-0 Downloads]$ conda install -c conda-forge pybind11  
Fetching package metadata ...........
Solving package specifications: .

Package plan for installation in environment /home/lynx/anaconda2/envs/pybind11-example:

The following NEW packages will be INSTALLED:

    pybind11: 2.1.1-py27_0 conda-forge

Proceed ([y]/n)? 

pybind11-2.1.1 100% |##########################################| Time: 0:00:00 231.13 kB/s

source deactivate
```
## Managing environments under Anaconda
You will now have at least four Anaconda environements: the original and three create as part of this exercise. You can list environemnts, and activate or deactive  the environments with the following commands:
```shell
conda info --envs
source activate <environment name>
(<environment-name>)[lynx@login-0-0] source deactivate
```
The following web page describes how to manage enviroments, https://conda.io/docs/using/envs.html