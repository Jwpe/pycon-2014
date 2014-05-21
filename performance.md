# Fast Python, Slow Python

## What is performance?

- It's making things go fast
- It's specialization

## Benchmarks

- Benchmarks are lies
- Any benchmark that doesn't look exactly like the application we're trying to mimic
will be wrong

## What is Python?

- Python vs CPython
- Python is an abstract language for which we've built implementation
- CPython is an implentation of Python

- Python is NOT:
    - Cython
 - C
    - RPython

## Optimizing Dynamically Typed Languages

- Dynamic languages do not **have** to be slow
- Monkey patching - how can we optimise?
    - Make assumptions
    - Do cheap checks on this assumption
- Slow != Hard to Optimise

## Pypy

- Pypy is another implementation of Python
- Uses Just-In-Time compiling(?)

## Python vs C

- How can we be as fast as C without static typing and immutability?

## Dictionaries vs Objects

- Objects have a fixed set of values
- Dicts have an arbitrary set of values
- Don't represent objects as dictionaries
- Classes are specialiseed, dictionaries aren't
- For the past 20 years, objects were seen as the same speed as dicts
- However dicts are much slower in modern Python, e.g Pypy, Python 3.2
- N.B. NamedTuples are fast

## Resources

- Zero-buffer


