---
layout: episode
title: API design
teaching: 20
exercises: 0
questions:
  - How can we create composable code units?
  - How can we improve the API to make it easier to understand and use?
objectives:
  - Discuss few guiding principles for API design.
keypoints:
  - Encapsulate.
  - Take some time to design an API.
  - Document and version your API.
---

## Coupling

### Strong coupling

![]({{ site.baseurl }}/img/strong-coupling.svg)


### Loose coupling

- Easier to reassemble
- Easier to understand

![]({{ site.baseurl }}/img/loose-coupling.svg)

---

## Cohesion

### Low cohesion

- Difficult to maintain, test, reuse, or even understand
- Non-cohesive code has unnecessary dependencies
- Swiss army knife modules

![]({{ site.baseurl }}/img/low-cohesion.svg)


### High cohesion

- Associated with robustness, reliability, reusability, and understandability
- Do one thing only and do it well
- API of cohesive code changes less over time
- Power of the Unix command line is a set of highly cohesive tools
- Microservices

![]({{ site.baseurl }}/img/high-cohesion.svg)

---

## Library guidelines (thanks to [Ole Martin Bjørndalen](https://github.com/olemb))

A good API:

- Lets me focus on my code, not figuring out what the library wants from me.
- Clear separation between library and my code so it doesn't take
  over my program. The library should be a tool or service I use, not
  one who comes over and lives at my house eats my food and never cleans
  up after itself.
- Makes common things easy and less common things possible.
- Hides all the cruft behind a simple API.
- Not require you to think about more than one thing at a time (or very few).
- Uses familiar data structures and concepts where possible.
- Does not expose inner plumbing that is irrelevant to you.
- Has good defaults - you shouldn't have to pass lots of options that
  are the same nearly every time.
- Follow the principle of least surprise.


### Other guidelines

- REPL friendly
- Tested on its own
- Built on its own
- Own development history (own Git repository)
- Compiled languages: provide C Interface
- Encapsulation
  - Hide internals
  - Interface exposed in a separate file
  - Expose the "what", hide the "how"
- Documentation
  - Separate the "what it can do" from "how is it implemented"
  - Documented API
  - Versioned API ([semantic](http://semver.org) or [sentimental](http://sentimentalversioning.org)
    or [romantic](https://github.com/jashkenas/backbone/issues/2888#issuecomment-29076249) versioning)

---

## We will discuss three API examples

- Stateless API
- API with state: new/compute/delete
- API with context

---

## Stateless API

### Example: [BLAS](https://en.wikipedia.org/wiki/Basic_Linear_Algebra_Subprograms)

```fortran
f = ddot(1000, vector_a, 1, vector_b, 1)
```

- From the client perspective there is no state.
- Sometimes libraries with "stateless" API have internal state (caching, memoization).

### Advantage

- We only need to consider this one call to understand and predict what will happen.

### Disadvantages

- The library may need to recompute expensive intermediates at every call.
- Often many arguments.

---

## API with state

- We split API into 3 phases/epochs
    - new/init
    - compute
    - delete/finalize

### Example

- [MPI](https://en.wikipedia.org/wiki/Message_Passing_Interface)

### Advantage

- We can avoid possibly costly initialization at each compute call.

### Disadvantages

- Order matters.
- We have to remember to clean up memory.
- Only one context at a time.

---

## API with state

Another example: we program our own bank:

```python
bank_new()

bank_deposit(100.0)
bank_deposit(100.0)

bank_withdraw(50.0)

my_balance = bank_get_balance()

bank_free()
```

Problem: We have to close the account before opening a new one.

It would be great to have more contexts:

```python
account1 = bank_new()

bank_deposit(account1, 100.0)
bank_deposit(account1, 100.0)

account2 = bank_new()

bank_deposit(account2, 200.0)
bank_deposit(account2, 200.0)

bank_withdraw(account1, 50.0)

balance1 = bank_get_balance(account1)
balance2 = bank_get_balance(account2)

bank_free(account1)
bank_free(account2)
```

### Example: FFTW

```c
fftw_complex in[N], out[N];
fftw_plan p;
...
p = fftw_create_plan(N, FFTW_FORWARD, FFTW_ESTIMATE);
...
fftw_one(p, in, out);
...
fftw_destroy_plan(p);
```

Discuss with the group other examples

---

## [Context-aware API in different languages](https://github.com/bast/context-api-example)

- Shows how to implement and use context-aware APIs in C++, Fortran, and Python
- Inspired by Armin Ronacher's
  ["Beautiful Native Libraries"](http://lucumr.pocoo.org/2013/8/18/beautiful-native-libraries/)

```
.
|-- CMakeLists.txt
|-- LICENSE
|-- README.md
|-- api
|   |-- cffi_helpers.py  (CFFI boilerplate code)
|   |-- example.h        (C interface)
|   `-- example.py       (Python interface)
|-- requirements.txt
|-- src
|   |-- bank.cpp         (C++ library)
|   |-- bank.f90         (Fortran library)
|   `-- bank.h           (C++ library)
`-- test
    |-- test.cpp         (C++ client)
    |-- test.f90         (Fortran client)
    `-- test.py          (Python client)
```

---

## There is no one-size-fits-all model

- Stateless works wonders when there is no need for a global initialization
- Exclusive state should work when creating more than one context does not make sense
- Context-aware works when multiple instances might be needed

---

## High-level and low-level API

### Scared of asking the client to pass 20 parameters every time?

- Offer a high-level API in addition to a low-level API
- Experts can access low-level API directly
- High-level convenience API is sufficient most of the time

```python
def deposit(account, amount):
    deposit_explicit(account=account,
                     amount=amount,
                     currency='EUR',
                     date=today(),
                     message='standard deposit',
                     ...)


def deposit_explicit(account, amount, currency, date, message, ...):
    ...
```
