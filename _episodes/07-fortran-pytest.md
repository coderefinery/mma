---
layout: episode
title: Testing Fortran with pytest
teaching: 40
exercises: 0
questions:
  - Write me.
objectives:
  - Write me.
keypoints:
  - Write me.
---

## Section

Hands-on example for testing a Fortran code with Pytest and deploying the test to Travis CI


Example Python code that we can start with:

```python
import random


def distance_to_origin_squared(x, y):
    return x*x + y*y


def approximate_pi(num_points):
    num_inside = 0
    for _ in range(num_points):
        x = random.uniform(0.0, 1.0)
        y = random.uniform(0.0, 1.0)
        if distance_to_origin_squared(x, y) < 1.0:
            num_inside += 1
    # we multiply by 4 to get the full circle
    # from the 4 segments
    return 4.0*num_inside/float(num_points)


for e in range(8):
    num_points = 10**e
    pi = approximate_pi(num_points)
    print(num_points, pi)
```
