---
layout: episode
title: Hands-on example using pybind11
teaching: 10
exercises: 15
questions:
  - What tool do you recommend to interface Python and C++?
objectives:
  - Unit testing and prototyping of C++ code.
keypoints:
  - pybind11 is the way to go to interface C++11 with Python.
---

## Example

In this example we wish to create a Python interface to a simple C++ function:

```cpp
int add(int i, int j) {
    return i + j;
}
```

as well as to a simple class. In this example C++ class we can create a pet,
give it a cute name, and take it for a walk and observe how our pet gets more
hungry after a long walk:

```cpp
class Pet
{
    public:
        Pet(const std::string &name, int hunger) : name(name), hunger(hunger) {}
        ~Pet() {}

        void go_for_a_walk() { hunger++; }
        const std::string &get_name() const { return name; }
        int get_hunger() const { return hunger; }

    private:
        std::string name;
        int hunger;
};
```

---

## Download and build the [example project](https://github.com/bast/pybind11-demo)

Note the `--recursive` when cloning the project. This is important since the
example project recursively clones the header library pybind11 which will
provide Python bindings after we have defined them.

```shell
$ git clone --recursive https://github.com/bast/pybind11-demo.git
$ cd pybind11-demo
$ mkdir build
$ cd build
$ cmake ..
$ make
```

You will hopefully see a result similar to this one:

```
$ make

Scanning dependencies of target example
[ 50%] Building CXX object CMakeFiles/example.dir/example.cpp.o
[100%] Linking CXX shared module example.cpython-36m-x86_64-linux-gnu.so
[100%] Built target example
```

---

## The interface definition

Before we test the Python interface, let us inspect how it is defined.
For this open [example.cpp](https://github.com/bast/pybind11-demo/blob/master/example.cpp#L25-L40):

```cpp
namespace py = pybind11;

PYBIND11_MODULE(example, m) {
    // optional module docstring
    m.doc() = "pybind11 example plugin";

    // define add function
    m.def("add", &add, "A function which adds two numbers");

    // bindings to Pet class
    py::class_<Pet>(m, "Pet")
        .def(py::init<const std::string &, int>())
        .def("go_for_a_walk", &Pet::go_for_a_walk)
        .def("get_hunger", &Pet::get_hunger)
        .def("get_name", &Pet::get_name);
}
```

Discuss the definitions that you see in `PYBIND11_MODULE` and how these map to
the function and the class.

Observe how little we have to write to define a Python interface with pybind11.

---

## Exercise 1: test the interface

Do this in the `build/` directory, either in the Python interpreter or a script (or a Jupyter notebook).

First test the add function:

```python
>>> from example import add
>>> add(2, 3)
5
```

Then also test the Pet class:

```python
>>> from example import Pet
>>> my_dog = Pet('Pluto', 5)
>>> my_dog.get_name()
'Pluto'
>>> my_dog.get_hunger()
5
>>> my_dog.go_for_a_walk()
>>> my_dog.get_hunger()
6
```

We are now able to call C++ code directly from Python!

---

## Exercise 2: extend the interface

Implement and test an additional function to make the pet bark or eat or sleep
or something else. If you extend the C++ code, you will need to recompile the
code (`make`).

Discuss why it could be useful to test or prototype your C++ code with Python using
pybind11.
