---
layout: episode
title: Motivation
teaching: 10
exercises: 0
questions:
  - Why is one programming language often not enough?
objectives:
  - Motivate why it can be beneficial to combine scripted and compiled languages.
keypoints:
  - Use the right tool for the right task and leverage tools.
---

## Why?

We want to combine the strengths of a scripting language, in our case Python, with
the strengths of a compiled language. A high-level scripting language is more
efficient for **prototyping** than a compiled language.  With a compiled language
our work process will be a compile-debug development cycle. This is time consuming,
but our compiled binaries will be more run-time efficient. We want both!

We may have **legacy code** that we want to make use of, but the nature of the code
base inhibits further development. Hence we want to move forward in a high-level
scripting language, but make use of previous code.

Our scripting language code base has **performance** problems. The time consumed to
solve certain task is too high. Consequently we want to move certain functions
to a compiled language.

These are all good reasons for extending the ability of our high-level
interpreter (the scripting language). 

```python
>>> import scipy
>>> import taylor
>>> scipy.sin(3.141592653/3)
0.86602540368613978
>>> taylor.sin(3.141592653/3,15)
0.8660254036861398
>>>
```

Here we import the well known [Scipy package](https://www.scipy.org), and call
the function `ts_sin()`. We also import the taylor library and call a
function `taylor.sin()` which returns the approximately same result as scipy.sin().

The taylor library is a shared library built from C++ source files, made
available to the python interpreter with the use of Cython or Pybind11.

## How does a scripting language talk to C/C++?
![Python and C/C++]({{ site.baseurl }}/img/python-c-002.png "Python and C/C++. Licenses CC BY 3.0"){:class="img-responsive"}

 Python, as most scripting language, has a foreign function
interface which defines how external functions commands can hook into the
interpreter ([Python c-api reference manual](https://docs.python.org/2/c-api/))

You will need to write a special wrapper that serves as a glue between the
C/C++function and the Python interpreter. The Python interpreter also needs to
known how to call the wrapper (the name,arguments. We could do this by
following the reference manual, but there exists tools/libraries which helps us
with this process.

Inspired by the [SWIG-3.0 documentation](http://www.swig.org/Doc3.0/)

