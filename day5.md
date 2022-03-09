# ??? Advanced Scala Day 5


## Potential Topics

- IO
- Variance
- Python 3.7+ equivalents to Scala
  - data class
  - pattern matching
- Creative coding (graphics)


## Python and Scala

Python equivalent to algebraic data types.

AFAIK (I'm definitely not a Python expert) Python has no true algebraic data types, but you can fake it (without any guarantees around correctness).

- Choosing between Scala and Python
- algebraic data types in Python

Scala
- performance. Python's performance is terrible.
- correctness. Type systems, FP.
- scalability. 

Python
- people know it.
- it's probably close to an imperative OO language that people already know if they don't already know it.
- no type system; don't need to express things in a way the compiler understands
- more data analysis / ML libraries
- more direct to work with for simple tasks

Rust
- fairly similar to Scala (algebraic data types, type classes)
- easier to link into Python than Scala, safer than C, safer and saner than C++
  - example: no use after free (automatic deterministic memory management)
- but not necessarily easy to learn

Python compilers
- Pypy / Mypy
- Numba (focused on numpy)


Algebraic data types in Python
- no true equivalent
- using dataclasses (and possibly pattern matching [3.10]) can get you a lot of the way there
- using Mypy to check types?
- maybe pylint as well?
