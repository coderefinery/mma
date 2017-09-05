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

## Why is one programming language often not enough?

|      | Python      | C                          | C++                                    | Fortran            |
|------|-------------|----------------------------|----------------------------------------|--------------------|
| Pros | prototyping | fast                       | low-level to high-level expressiveness | math               |
|      | readability | portable                   | data structures                        | speed              |
|      | libraries   |                            |                                        |                    |
|------|-------------|----------------------------|----------------------------------------|--------------------|
| Cons | speed       | low-level                  | complexity                             | lack of containers |
|      | type system | explicit memory management | lots of boilerplate code               | input parsing      |

Some cons are sometimes pros. This is not a flame-war.
The point here is that different languages have their own strengths and weaknesses.

---

## Motivation for this tutorial

- Each language has strengths and weaknesses, combine the strengths.
- To speed up Python code: move the bottleneck outside of Python.
- To simplify compiled code: express code that is not efficiency-critical in Python.
- To combine libraries or legacy code written in different languages: sometimes others have chosen the language.
- To unit test compiled code with Python: simple and you can write tests without recompiling the code.
- To package compiled code with Python: [PyPI](https://pypi.org) is awesome.

Can you think of more reasons?

In this tutorial we will discuss and present how to glue different languages
together - buckle up!
