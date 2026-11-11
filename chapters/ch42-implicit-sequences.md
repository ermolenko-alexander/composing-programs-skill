# Section 4.2: Implicit Sequences

## Core Idea
A sequence can be represented without each element being stored explicitly in memory: construct an object that provides access to all elements of a sequential dataset without computing each value in advance — compute elements on demand. Computing values on demand, rather than retrieving them from an existing representation, is an example of **lazy computation**: any program that delays the computation of a value until that value is needed. Python's iterator/iterable protocols, generators, and lazily computed streams all implement this one idea with different trade-offs.

The motivating example is `range`: `r = range(10000, 1000000000)` does not store 999,990,000 integers — only the endpoints. `r[45006230]` is computed as 10,000 + 45,006,230 = 45,016,230. Ranges can do this because the sequence is uniform; iterators generalize lazy generation to a much broader class of series because they only need to compute the *next* element in order, not arbitrary elements.

## Formal Protocols (verbatim-faithful)

### Iterator interface
- An **iterator** is an object that provides sequential access to values, one by one. The abstraction has two components: a mechanism for retrieving the next element, and a mechanism for signaling that the end of the sequence has been reached and no further elements remain.
- `__next__` returns the next element of the underlying series. In response, an iterator may perform arbitrary computation to retrieve or compute it. Calls to `__next__` make a **mutating change** to the iterator: they advance its position.
- The end of the series is signaled by raising a `StopIteration` exception during a call to `__next__`. Once raised, the iterator continues to raise it from then on; typically an iterator is not reset — a new instance is created for a new pass.
- The built-ins: `iter(x)` invokes `x.__iter__()`; `next(it)` invokes `it.__next__()`. `StopIteration` can be handled with a `try` statement.
- The Python docs suggest an iterator also have an `__iter__` method that returns the iterator itself, so that **all iterators are iterable** (required to use an iterator in a `for` loop).
- An iterator maintains local state for its position: two separate iterators over the same iterable track independent positions, but two names for the same iterator share position. `iter(iterator)` returns that same iterator, **not a copy** — so you can call `iter` on any value without caring whether it is a container or already an iterator.
- A `__next__` that never raises `StopIteration` represents an **infinite series** (e.g. `Positives`).

### Iterable interface
- Any value that can produce iterators is an **iterable**; in Python, anything that can be passed to `iter`. Iterables include strings, tuples, sets, dictionaries — and iterators themselves.
- The interface is a single message, `__iter__`, that returns an iterator. If an iterable returns a **fresh iterator instance on each `__iter__` call**, it can be iterated multiple times.
- The iterable represents the data itself (fixed); the iterator tracks progress (mutable).
- Even unordered collections (dicts, sets) must define an ordering over their contents when producing iterators.

### For-statement execution rule
```
for <name> in <expression>:
    <suite>
```
To execute: Python evaluates the header `<expression>`, which must yield an iterable value. The `__iter__` method is invoked on it. Until a `StopIteration` exception is raised, Python repeatedly invokes `__next__` on that iterator, binds the result to `<name>`, then executes `<suite>`. Equivalent desugaring:
```python
>>> items = counts.__iter__()
>>> try:
        while True:
            item = items.__next__()
            print(item)
    except StopIteration:
        pass
```

### Generator execution rules
- A **generator** is an iterator returned by a **generator function** — a function that uses `yield` statements instead of `return` to produce elements of a series.
- Calling a generator function does not return a yielded value; it returns a generator (a type of iterator, `<class 'generator'>`) with `__iter__` and `__next__` methods. **The body does not execute at all until the first `__next__`.**
- Each `__next__` call continues execution of the function body from wherever it left off, until the next `yield` executes; it then pauses and returns the yielded value. `yield` statements **do not destroy the newly created environment — they preserve it for later**: the values of all bound names in the function's scope persist across `__next__` calls.
- The generator raises `StopIteration` whenever its generator function returns.
- Generators do not use object attributes to track progress through a series; they control the execution of the function itself.

### Stream (lazy linked list) protocol
- A stream responds to `first` and `rest`, like `Link`; `rest` is itself a `Stream` — but it is computed only when looked up, not stored in advance.
- A stream stores a `compute_rest` function. Its essential properties: **it takes no arguments, and returns a `Stream` or `Stream.empty`.**
- On first access of the `rest` property, `self._rest = self._compute_rest()` runs and `self._compute_rest` is set to `None`: the function is **called exactly once, then discarded** (cached/memoized).

## Code Examples

### Iterator basics and shared position
```python
>>> primes = [2, 3, 5, 7]
>>> iterator = iter(primes)
>>> next(iterator)
2
>>> r = range(3, 13)
>>> s = iter(r)    # 1st iterator over r
>>> next(s); next(s)        # 3, 4
>>> t = iter(r)    # 2nd iterator over r — independent position
>>> next(t); next(t)        # 3, 4
>>> u = t          # alternate name for the 2nd iterator — shared position
>>> next(u); next(u)        # 5, 6
>>> next(s)        # 5  (1st iterator unaffected)
>>> next(t)        # 7  (t shares position with u)
>>> v = iter(t)    # iter on an iterator returns the SAME iterator
>>> next(v); next(u); next(t)   # 8, 9, 10
```

### Hand-written iterator class
```python
>>> class LetterIter:
        """An iterator over letters of the alphabet in ASCII order."""
        def __init__(self, start='a', end='e'):
            self.next_letter = start
            self.end = end
        def __next__(self):
            if self.next_letter == self.end:
                raise StopIteration
            letter = self.next_letter
            self.next_letter = chr(ord(letter)+1)
            return letter
```
An infinite iterator never raises `StopIteration`:
```python
>>> class Positives:
        def __init__(self):
            self.next_positive = 1
        def __next__(self):
            result = self.next_positive
            self.next_positive += 1
            return result
```

### Iterable class producing fresh iterators
```python
>>> class Letters:
        def __init__(self, start='a', end='e'):
            self.start = start
            self.end = end
        def __iter__(self):
            return LetterIter(self.start, self.end)
```

### Generator function replacing the class
```python
>>> def letters_generator():
        current = 'a'
        while current <= 'd':
            yield current
            current = chr(ord(current)+1)
>>> letters = letters_generator()
>>> type(letters)
<class 'generator'>
>>> letters.__next__()
'a'
```
After `'d'`, the next `__next__` raises `StopIteration`.

### Iterable with a generator `__iter__`
```python
>>> class LettersWithYield:
        def __init__(self, start='a', end='e'):
            self.start = start
            self.end = end
        def __iter__(self):
            next_letter = self.start
            while next_letter < self.end:
                yield next_letter
                next_letter = chr(ord(next_letter)+1)
```
Each `__iter__` invocation returns a new generator that starts a fresh pass.

### Stream class and operations
```python
>>> class Stream:
        """A lazily computed linked list."""
        class empty:
            def __repr__(self):
                return 'Stream.empty'
        empty = empty()
        def __init__(self, first, compute_rest=lambda: empty):
            assert callable(compute_rest), 'compute_rest must be callable.'
            self.first = first
            self._compute_rest = compute_rest
        @property
        def rest(self):
            """Return the rest of the stream, computing it if necessary."""
            if self._compute_rest is not None:
                self._rest = self._compute_rest()
                self._compute_rest = None
            return self._rest
        def __repr__(self):
            return 'Stream({0}, <...>)'.format(repr(self.first))

>>> def integer_stream(first):
        def compute_rest():
            return integer_stream(first+1)
        return Stream(first, compute_rest)

>>> def map_stream(fn, s):
        if s is Stream.empty:
            return s
        def compute_rest():
            return map_stream(fn, s.rest)
        return Stream(fn(s.first), compute_rest)

>>> def filter_stream(fn, s):
        if s is Stream.empty:
            return s
        def compute_rest():
            return filter_stream(fn, s.rest)
        if fn(s.first):
            return Stream(s.first, compute_rest)
        else:
            return compute_rest()

>>> def first_k_as_list(s, k):
        first_k = []
        while s is not Stream.empty and k > 0:
            first_k.append(s.first)
            s, k = s.rest, k-1
        return first_k
```
Pattern note: `map_stream`/`filter_stream` exhibit the common stream-processing pattern — a locally defined `compute_rest` recursively applies the processing function to the rest whenever the rest is computed. In `filter_stream`, if the filter rejects the first element the rest is computed immediately (possibly repeatedly) until a valid `first` is found.

## Worked Examples

### 1. Lazy `map` — observing deferred computation
```python
>>> def double_and_print(x):
        print('***', x, '=>', 2*x, '***')
        return 2*x
>>> s = range(3, 7)
>>> doubled = map(double_and_print, s)  # double_and_print not yet called
>>> next(doubled)                       # called once
*** 3 => 6 ***
6
>>> next(doubled)
*** 4 => 8 ***
8
>>> list(doubled)                       # called twice more
*** 5 => 10 ***
*** 6 => 12 ***
[10, 12]
```
`map` is lazy: calling it creates an iterator and performs no computation. `filter`, `zip`, and `reversed` also return iterators.

### 2. `Letters`/`LetterIter` — iterable vs iterator
`b_to_k = Letters('b', 'k')`; `first_iterator = b_to_k.__iter__()` yields `'b'`, `'c'`; then `second_iterator = iter(b_to_k)` starts over at `'b'` while `first_iterator` continues at `'d'`, `'e'`. The `Letters` instance never changes; each `LetterIter` mutates with every `next`. The iterator tracks progress through sequential data; the iterable represents the data itself. Built-ins compose: `caps = map(lambda x: x.upper(), b_to_k)`; `next(caps)` → `'B'`.

### 3. `all_pairs` — why multiple passes need an iterable
```python
>>> def all_pairs(s):
        for item1 in s:
            for item2 in s:
                yield (item1, item2)
>>> list(all_pairs([1, 2, 3]))
[(1, 1), (1, 2), (1, 3), (2, 1), (2, 2), (2, 3), (3, 1), (3, 2), (3, 3)]
```
The inner loop re-iterates `s` once per outer element — this only works because a list's `__iter__` returns a fresh iterator each time. `list(all_pairs(LettersWithYield()))[:5]` gives `[('a','a'), ('a','b'), ('a','c'), ('a','d'), ('b','a')]`. Passing a plain iterator instead would exhaust on the first outer element.

### 4. Stream vs Link, and the sieve of Eratosthenes
```python
>>> r = Link(1, Link(2+3, Link(9)))
>>> s = Stream(1, lambda: Stream(2+3, lambda: Stream(9)))
>>> r.rest.first   # 5 — stored
>>> s.rest.first   # 5 — computed on demand, the first time requested
>>> s.rest
Stream(5, <...>)
```
Squares: `m = map_stream(lambda x: x*x, integer_stream(3))`; `first_k_as_list(m, 5)` → `[9, 16, 25, 36, 49]`.

Primes — filter a stream of integers to remove all multiples of its first element; by successively filtering with each prime, all composites are removed:
```python
>>> def primes(pos_stream):
        def not_divible(x):
            return x % pos_stream.first != 0
        def compute_rest():
            return primes(filter_stream(not_divible, pos_stream.rest))
        return Stream(pos_stream.first, compute_rest)
>>> prime_numbers = primes(integer_stream(2))
>>> first_k_as_list(prime_numbers, 7)
[2, 3, 5, 7, 11, 13, 17]
>>> prime_numbers.first
2
```
Streams contrast with iterators: they can be passed to pure functions multiple times and yield the same result each time. The primes stream is not "used up" by converting a prefix to a list — `first` is still 2 afterwards.

## Reference Table: Iterator vs Generator vs Stream

| | Iterator class | Generator | Stream |
|---|---|---|---|
| State kept in | instance attributes (`self.next_letter`) | the suspended function's frame/environment | `first` + cached `_rest` |
| Advance | `__next__` runs method body | `__next__` resumes body until next `yield` | access `rest` property, runs `compute_rest` once |
| End signal | raise `StopIteration` | function returns → `StopIteration` | `rest` is `Stream.empty` |
| Reusable? | no — one pass, then always `StopIteration` | no — one pass | yes — not consumed; same result for pure functions every time |
| Infinite series | `__next__` never raises | `while True: yield ...` | recursive `compute_rest` (`integer_stream`) |
| Best for | simple state; interop | complex iteration logic where saving your place manually is hard | shareable, re-traversable lazy sequences; functional recursion |

## Key Concepts
- **Lazy computation**: delaying computation of a value until it is needed.
- **Iterator**: sequential access, one value at a time; mutable position.
- **Iterable**: anything passable to `iter`; the fixed data representation.
- **StopIteration**: the end-of-series signal; persistent once raised.
- **Generator / generator function**: `yield`-based iterator; body suspended/resumed by the interpreter.
- **Sequential vs random access**: iterators give only sequential access, "often sufficient for data processing applications".
- **Lazy built-ins**: `map`, `filter`, `zip`, `reversed` return iterators; nothing runs until pulled.
- **Stream**: lazily computed linked list; `compute_rest` runs once and is cached.
- **compute_rest contract**: zero arguments; returns `Stream` or `Stream.empty`.

## Mental Models
- An iterator is a bookmark; the iterable is the book. One book, many bookmarks; two names for one bookmark move together.
- A generator is a paused function: `yield` is "return, but keep my frame alive for next time."
- A `Stream` is a `Link` whose `rest` is a promise, kept at most once.
- Streams are the functional/persistent flavor of laziness; iterators/generators are the imperative/consumable flavor.

## Anti-patterns / Common Errors
- **Mutating a dict's structure during iteration**: adding or removing a key invalidates all its iterators (`RuntimeError: dictionary changed size during iteration`); future iterators may show arbitrary order changes. Changing an existing key's value is fine.
- **Reusing an exhausted iterator**: after `StopIteration` it keeps raising; create a new iterator (or pass an iterable with a fresh `__iter__`, as `all_pairs` requires).
- **Expecting `iter(iterator)` to copy**: it returns the same iterator, sharing position.
- **Expecting a generator to run at call time**: no body statement executes until the first `__next__`.
- **Hand-rolling `self.current` state for complex iterations**: use a generator and let the interpreter save your place.
- **Passing a non-callable as `compute_rest`**: the class asserts `callable(compute_rest)` — the rest must be a delayed computation, not a value.

## Key Takeaways
1. Represent large/unbounded sequences implicitly; compute elements on demand.
2. Distinguish iterable (data, `__iter__`) from iterator (position, `__next__` + `StopIteration`).
3. A `for` statement is `__iter__` + repeated `__next__` inside `try/except StopIteration`.
4. Prefer generator functions over iterator classes for nontrivial iteration state.
5. Make `__iter__` a generator function to get a cheaply re-iterable object.
6. Streams give persistent, functional laziness: cached one-shot `compute_rest`, never "used up".
7. `map`/`filter`/`zip`/`reversed` are lazy — pull to compute.
8. Infinite series are fine: a `__next__` that never stops, or a recursive `compute_rest`.

## Connects To
- **Section 4.1**: realizes the pipeline-over-streams organization.
- **Chapter 2 (sequence interface, `Link`)**: `Stream` is a lazy `Link`; built-in sequences implement the iterable interface.
- **Chapter 1 (higher-order functions)**: `map_stream`/`filter_stream` are HOFs whose laziness lives in locally defined closures.
- **Section 4.7**: MapReduce consumes and emits sequential streams via iterators/generators.
