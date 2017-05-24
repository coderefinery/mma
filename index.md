---
layout: lesson
permalink: /
---

# Mixed Martial Arts: Interfacing Fortran, C, C++, and Python for Great Good!

Write me ...


## Prerequisites

1. You will need
 * a C++ compiler which is C++11 compliant
 * CMake
 * Anaconda2
 * Make three different environments in Anaconda with different some different
 packages installed in each environment. Instructions follows



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

## Creating the first work environment - the swig-example
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
## Creating the second work environment - the boost-example
We create a new Anaconda enviroment for use with the Boost-example. Here we install the Boost Library and scipy (for comparision). Remember to deactivate your current conda environment before creating the Boost environment `source deactivate`:

```shell
[lynx@~]$ conda create --name boost-example
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

```


## Managing environments under Anaconda

You will now have at least four Anaconda environements: the original and three create as part of this exercise. You can list environemnts, and activate or deactive  the environments with the following commands:
```shell
conda info --envs
source activate <environment name>
(<environment-name>)[lynx@login-0-0] source deactivate
```
The following web page describes how to manage enviroments, https://conda.io/docs/using/envs.html



