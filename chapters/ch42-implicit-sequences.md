# Section 4.2: Implicit Sequences

## Core Idea
A sequence can be represented without storing each element: compute values on demand ("lazy computation"). Python's iterator/iterable protocols, generators, and lazily-computed streams all implement this one idea with different trade-offs.

## Frameworks Introduced
- **Lazy computation**: any program that delays computing a value until it is needed (e.g. `range(10000, 1000000000)` stores only endpoints and computes `r[i]` on demand).
  - When to use: large or unbounded sequences; expensive per-element work that may never be requested.
  - How: store how to compute elements rather than the elements themselves.
- **Iterator abstraction**: two components — a mechanism to retrieve the next element, and a mechanism to signal the end (`StopIteration`).
  - When to use: single-pass sequential access ("sequential access" vs "random access").
  - How: `iter(container)` to get an iterator, `next(it)` to advance; handle `StopIteration` with `try`.
- **Iterable interface**: an object is iterable if its `__iter__` method returns an iterator; a fresh iterator per `__iter__` call allows multiple passes.
- **Iterator interface**: `__next__` returns the next element (mutating the iterator's position) and raises `StopIteration` at the end; iterators also define `__iter__` returning themselves, so all iterators are iterable.
- **Generators and yield statements**: a generator function uses `yield` instead of `return`; calling it returns a generator (a kind of iterator) whose `__next__` resumes the function body until the next `yield`, preserving the local environment across calls.
  - When to use: when hand-writing `__next__` state tracking (`self.current`) would be awkward for a complex iteration.
- **Creating iterables with yield**: make `__iter__` itself a generator function; every call starts a fresh pass (e.g. `LettersWithYield`).
- **Streams (lazily computed linked lists)**: a `Stream` has a `first` and a `compute_rest` zero-argument function; `rest` is a property that calls `compute_rest` once, caches the result in `_rest`, and discards the function.
  - When to use: when you need a persistent, re-traversable lazy sequence that pure functions can consume repeatedly without "using it up".

## Key Concepts
- **Iterator**: an object providing sequential access to values one by one via `next`.
- **Iterable**: any value that can be passed to `iter` (strings, tuples, sets, dicts, iterators themselves).
- **StopIteration**: the exception that signals a series is exhausted; once raised, the iterator keeps raising it.
- **Generator**: the iterator returned by a generator function; body does not execute until the first `__next__`.
- **Generator function**: a function containing `yield`; returns a generator rather than a value.
- **Lazy built-ins**: `map`, `filter`, `zip`, `reversed` return iterators and defer all computation until elements are requested.
- **Random vs sequential access**: iterators give only sequential access, which is usually sufficient for data processing.
- **Stream**: a lazily computed linked list; `rest` is computed on first lookup and cached.
- **compute_rest function**: takes no arguments; returns a `Stream` or `Stream.empty`.

## Mental Models
- Think of an iterator as a mutable position in an underlying series; two iterators over the same iterable are independent, but two names for one iterator share position.
- Think of an iterable as the data itself and the iterator as a bookmark; one fixed iterable can produce many bookmarks.
- Use generators when saving your place in a computation is harder than letting the interpreter suspend/resume it for you.
- Think of a `Stream` as a `Link` whose `rest` is a promise: streams are not "used up", unlike iterators, so they can be passed to pure functions repeatedly with the same result.

## Anti-patterns
- **Mutating a dict's structure during iteration**: adding/removing keys invalidates all its iterators (`RuntimeError: dictionary changed size during iteration`); changing an existing key's value is fine.
- **Reusing an exhausted iterator**: iterators make one pass; create a new iterator (or an iterable with fresh `__iter__`) for another pass — e.g. `all_pairs` needs an iterable, not an iterator.
- **Expecting `iter(iterator)` to copy**: it returns the same iterator, sharing position.
- **Storing manual state in `__next__` for complex iterations**: hard to maintain; use a generator instead.

## Code Examples
```python
>>> def letters_generator():
        current = 'a'
        while current <= 'd':
            yield current
            current = chr(ord(current)+1)
>>> for letter in letters_generator():
        print(letter)
a
b
c
d
```
- **What it demonstrates**: a generator function replaces an iterator class — `yield` suspends and preserves the environment; execution resumes on each `__next__`.

```python
class Stream:
    """A lazily computed linked list."""
    def __init__(self, first, compute_rest=lambda: empty):
        assert callable(compute_rest), 'compute_rest must be callable.'
        self.first = first
        self._compute_rest = compute_rest
    @property
    def rest(self):
        if self._compute_rest is not None:
            self._rest = self._compute_rest()
            self._compute_rest = None
        return self._rest
```
- **What it demonstrates**: lazy evaluation with memoization — `compute_rest` runs exactly once, then is discarded.

## Worked Example
Sieve of Eratosthenes on an infinite stream. `integer_stream(first)` builds `Stream(first, lambda-ish compute_rest)` recursively; `map_stream`/`filter_stream` wrap a processing function in a locally defined `compute_rest` applied when `rest` is requested. Then:

```python
def primes(pos_stream):
    def not_divible(x):
        return x % pos_stream.first != 0
    def compute_rest():
        return primes(filter_stream(not_divible, pos_stream.rest))
    return Stream(pos_stream.first, compute_rest)
```

`first_k_as_list(primes(integer_stream(2)), 7)` gives `[2, 3, 5, 7, 11, 13, 17]` — each prime successively filters its multiples out of the infinite tail, and only the requested prefix is ever computed. Afterwards `prime_numbers.first` is still 2: streams are not consumed.

## Key Takeaways
1. Represent large/unbounded sequences implicitly; compute elements on demand.
2. Distinguish iterable (the data, `__iter__`) from iterator (the position, `__next__` + `StopIteration`).
3. A `for` statement is just `__iter__` + repeated `__next__` inside a `try/except StopIteration`.
4. Prefer generator functions over hand-rolled iterator classes for nontrivial iteration state.
5. Make `__iter__` a generator function to get a re-iterable object cheaply.
6. Use streams when a lazy sequence must be shareable and repeatedly traversable (functional, cached).
7. `map`/`filter` are lazy: nothing runs until you pull elements.

## Connects To
- **Section 4.1**: realizes the pipeline-over-streams program organization.
- **Section 2 (sequence interface, Link)**: `Stream` is a lazy `Link`; built-in sequences implement the iterable interface.
- **Section 4.7**: MapReduce consumes and emits sequential streams via iterators/generators.
