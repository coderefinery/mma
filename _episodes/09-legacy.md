---
layout: episode
title: Roadmap for migrating and modularizing legacy code
teaching: 10
exercises: 0
questions:
  - Big untested legacy monolith code in front of you - what now?
objectives:
  - Offer a step-by-step recipe for migrating and modularizing legacy code.
keypoints:
  - Introduce testing early as a safeguard.
  - Use code coverage analysis tools.
  - Cut the code at important interfaces.
  - Build and test modules separately to identify hidden dependencies.
---

## Section

TODO: show an example/illustration for each step

- Add end-to-end test
- If possible, add unit tests
- Test coverage, iterate until test coverage is sufficient
- Define interfaces
- Isolate modules behind interfaces
- Refine interfaces
- Localize global data
- Minimize dependencies
- Maximize cohesion
- Document interfaces
- Build modules separately into libraries
- Test modules separately
- Outsource modules into own repositories
- Include external repositories with the help of CMake and possibly git submodules/subtrees
