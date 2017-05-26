---
layout: episode
title: Fortran, C and C++ talking to each other
teaching: 20
exercises: 0
questions:
  - Write me.
objectives:
  - Write me.
keypoints:
  - Write me.
---

## Learning goals

- Calling Fortran libraries from C(++)
- Calling C(++) libraries from Fortran

---

## Example

Imagine you are on a desert island and wish to compute pi.
You have a computer with you with compilers installed but no math libraries and no Wikipedia.

**SPOILER BELOW**

<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>

Here is one way of doing it - throwing darts (generating random points in an
interval and checking whether the points are within a given radius):

![]({{ site.baseurl }}/img/darts.svg)

Discuss briefly how you can approximate pi from such an experiment and how you would program it.

In this workshop we will implement this example in 3 different languages (C++,
Fortran, Python) and demonstrate how to call this functionality across
languages.

Later we will combine these 3 implementations in an example Python package.

---

## Exercise 1

Test the following Python code:

```python
import random


def _distance_to_origin_squared(x, y):
    return x*x + y*y


def approximate_pi(num_points):
    random.seed(0)

    num_inside = 0
    for _ in range(num_points):
        x = random.uniform(0.0, 1.0)
        y = random.uniform(0.0, 1.0)
        if _distance_to_origin_squared(x, y) < 1.0:
            num_inside += 1

    # we multiply by 4 to get the full circle
    # from the 4 segments
    return 4.0*num_inside/float(num_points)
```

Does it converge to the right number?
