---
layout: episode
title: Memory allocation for arrays that pass the API
teaching: 10
exercises: 0
questions:
  - Where and how should we allocate memory?
#objectives:
# - Learn the tools to create a Python interface to almost any C API.
# - Obtain a recipe for testing compiled libraries with Python.
keypoints:
  - Allocate API arrays client-side.
  - Write a thin Python layer around them.
---

## Memory allocation for arrays that pass the API

![]({{ site.baseurl }}/img/cffi/terminator.jpg)

### Library side

Advantage:

- Caller does not need to know the array sizes *a priori*.

Disadvantage:

- Risk of a memory leak (garbage collection on the Python side).


### Client side (recommended)

Advantages:

- Let the caller decide how memory is allocated.
- Reduced risk of a memory leak.

Disadvantage:

- You need to know array sizes.
- Requires extra layer on the Python side.


### Example

Consider these two functions:

```cpp
double *get_array_leaky(const int len)
{
    double *array = new double[len];

    for (int i = 0; i < len; i++)
    {
        array[i] = (double)i;
    }

    return array;
}

void get_array_safe(const int len, double array[])
{
    for (int i = 0; i < len; i++)
    {
        array[i] = (double)i;
    }
}
```

How can we use the second one?

```python
from cffi import FFI

def get_array_safe(length):
    ffi = FFI()
    lib = ffi.dlopen(...)

    array = ffi.new("double[]", length)
    lib.get_array_safe(length, array)

    return array
```

- Demonstration: [Client-side or library-side memory allocation
  with Python CFFI](https://github.com/bast/cffi-mem-alloc-example).
- You need to hold on to the object for as long as you need the memory on the library-side.


### Questions

- How can you make the API function return a Python list or dict?
- Can you implement generators?
