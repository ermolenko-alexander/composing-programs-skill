# Section 4.7: Distributed Data Processing

## Core Idea
Distributed systems are often used to collect, access, and manipulate large data sets — e.g. database systems operating over data stored across multiple machines, where no single machine holds all data needed to answer a query. The typical big-data scenario: a data set too large to be processed by a single machine is distributed among many machines, each processing a portion; results must be aggregated across machines. The coordinating framework is **MapReduce**: express the application in terms of **pure functions** used to *map* over a large dataset and to *reduce* the mapped sequences of values into a final result, and let the framework own all communication, coordination, and failure handling.

Functional programming is used to maximal advantage: MapReduce **requires** that map and reduce be pure functions. A program expressed only in terms of pure functions has considerable flexibility in execution — sub-expressions can be computed **in arbitrary order and in parallel** without affecting the final result — so a MapReduce application evaluates many pure functions in parallel, reordering computations for efficient distributed execution.

## The Framework

### Separation of concerns (the principal advantage)
MapReduce enforces a separation between two parts of a distributed data processing application:
1. The map and reduce functions that process data and combine results.
2. The communication and coordination between machines.

The coordination mechanism handles machine failures, network failures, and progress monitoring. None of that complexity is exposed to the application developer: building a MapReduce application only requires specifying the functions in (1); the challenges of distributed computation are hidden via abstraction.

### The three-step computation
Input: a large, **unordered stream** of input values of arbitrary type (e.g. each input a line of text in some vast corpus). Computation proceeds in three steps:
1. A **map function** is applied to each input, outputting zero or more intermediate key-value pairs of an arbitrary type.
2. All intermediate key-value pairs are **grouped by key**, so that pairs with the same key can be reduced together.
3. A **reduce function** combines the values for a given key k; it outputs zero or more values, each associated with k in the final output.

### Tasks and phases
- A **map task** applies the map function to some subset of the input data and outputs intermediate key-value pairs.
- A **reduce task** sorts and groups key-value pairs by key, then applies the reduce function to the values for each key.
- All communication between map and reduce tasks, and the grouping of intermediate pairs by key, is handled by the framework.
- To utilize multiple machines: multiple mappers run in parallel in the **map phase**; multiple reducers run in parallel in the **reduce phase**; in between, the **sort phase** groups key-value pairs by sorting them, so all pairs with the same key are adjacent.

## Local Implementation with Unix Pipes

Unix creates an abstraction barrier between user programs and hardware and lets one program consume another's output. Kernighan and Pike: "The power of a system comes more from the relationships among programs than from the programs themselves."

A Python file becomes a Unix program by a first-line comment naming the interpreter. **Standard input** is the iterable `sys.stdin` (yields string lines); **standard output** is `sys.stdout`, written by `print`. Example — reverse each input line:
```python
#!/usr/bin/env python3

import sys

for line in sys.stdin:
    print(line.strip('\n')[::-1])
```
```
$ chmod u+x rev.py                      # mark executable
$ nslookup 170.149.172.130 | ./rev.py   # pipe: output of one program into another
moc.semityn.www
$ cat rev.py | ./rev.py                 # cat outputs file contents
3nohtyp vne/nib/rsu/!#
...
```

These tools implement a basic MapReduce framework with a single map task and single reduce task:
```
$ cat input | ./mapper.py | sort | ./reducer.py
```
`sort` plays the sort phase: it makes same-key pairs adjacent.

## Code Examples: the vowel-counting application

Map function — key is a vowel, value is a count; zero counts omitted:
```python
#!/usr/bin/env python3
# count_vowels_mapper.py

import sys
from mr import emit

def count_vowels(line):
    """A map function that counts the vowels in a line."""
    for vowel in 'aeiou':
        count = line.count(vowel)
        if count > 0:
            emit(vowel, count)

for line in sys.stdin:
    count_vowels(line)
```

Reduce function — Python's built-in `sum` over each key's value iterator:
```python
#!/usr/bin/env python3
# sum_reducer.py

import sys
from mr import values_by_key, emit

for key, value_iterator in values_by_key(sys.stdin):
    emit(key, sum(value_iterator))
```

The companion `mr` module provides `emit` (emit a key-value pair) and grouping of values that share a key, plus an interface to the Hadoop distributed implementation.

## Worked Example: counting vowels in a haiku
Input file `haiku.txt`:
```
Google MapReduce
Is a Big Data framework
For batch processing
```
Local execution:
```
$ cat haiku.txt | ./count_vowels_mapper.py | sort | ./sum_reducer.py
'a'   6
'e'   5
'i'   2
'o'   5
'u'   1
```
Trace: the mapper emits per-line pairs such as `('o', 3)`, `('e', 3)` for line 1, `('a', 4)`, `('i', 2)` for line 2, etc.; `sort` brings all `'a'` pairs together, all `'e'` pairs together; the reducer sums each key's iterator. The **identical** mapper and reducer files then run distributed:
```
$ python3 mr.py run count_vowels_mapper.py sum_reducer.py [input] [output]
```
where `[input]` and `[output]` are directories in the Hadoop distributed file system (after installing Hadoop, setting `HADOOP` to the installation root, and copying text files into the Hadoop file system).

## Distributed Implementation: Hadoop
**Hadoop** is an open-source implementation of the MapReduce framework that executes applications on a cluster of machines, distributing input data and computation for efficient parallel processing. Its **streaming interface** allows arbitrary Unix programs to define the map and reduce functions — `count_vowels_mapper.py` and `sum_reducer.py` can be used directly with a Hadoop installation on large corpora.

Advantages over the local pipeline:
1. **Speed**: map and reduce functions are applied in parallel by different tasks on different machines running simultaneously.
2. **Fault tolerance**: when a task fails for any reason, its result can be recomputed by another task to complete the overall computation.
3. **Monitoring**: a user interface for tracking the progress of a MapReduce application.

| | Unix-pipe version | Hadoop |
|---|---|---|
| Map/reduce tasks | one of each | many, parallel, on many machines |
| Grouping | `sort` command | framework sort phase |
| Failure handling | none | failed tasks recomputed elsewhere |
| Monitoring | none | progress UI |
| Programs | `mapper.py`, `reducer.py` | the same files, via streaming |

## Key Concepts
- **Map function / map task**: per-input emission of zero or more intermediate key-value pairs; a task applies it to a subset of inputs.
- **Reduce function / reduce task**: per-key combination of values; a task sorts, groups, and reduces.
- **Map / sort / reduce phases**: parallel mappers → group by sorted key → parallel reducers.
- **emit / values_by_key**: the `mr` module's output and grouping helpers.
- **Standard input / standard output / pipe**: `sys.stdin` (iterable of lines), `print`, and `|` composing Unix programs.
- **Streaming interface**: Hadoop's mechanism for using any Unix program as mapper or reducer.
- **Purity requirement**: the contract that licenses reordering, parallelism, and re-execution.

## Mental Models
- MapReduce is functional programming industrialized: purity buys the freedom to reorder and parallelize.
- `mapper | sort | reducer` is the entire framework in one shell line; Hadoop is the same pipeline plus parallelism, fault tolerance, and monitoring.
- The key is the unit of aggregation: choose keys so everything that must be combined shares one.
- The abstraction barrier here mirrors Unix's own: programs compose through streams without knowing each other's internals.

## Anti-patterns / Common Errors
- **Impure map or reduce functions**: side effects or order dependence break the framework's freedom to reorder, parallelize, and — critically — re-execute failed tasks (fault tolerance literally reruns your function).
- **Relying on input order**: the input is an *unordered* stream; the only ordering guarantee is that same-key pairs are adjacent after the sort phase.
- **Hand-rolling machine coordination**: failure handling, communication, and progress tracking belong to the framework; writing them yourself forfeits the abstraction.
- **Emitting zero-count/no-op pairs**: the mapper may emit zero pairs for an input — use that rather than emitting junk to filter later.

## Key Takeaways
1. Express distributed data processing as pure map and reduce functions; the framework does the rest.
2. The three-step model (map → group by key → reduce) covers a huge class of batch computations.
3. Prototype locally with `cat input | ./mapper.py | sort | ./reducer.py`; the same files scale to a cluster via Hadoop streaming.
4. Purity is the contract that makes parallel execution and fault-tolerant re-execution safe.
5. Framework benefits: parallel speed, recomputation on failure, progress monitoring — none exposed to the developer.
6. Unix philosophy underlies it: the system's power comes from relationships among programs.

## Connects To
- **Section 4.2**: `sys.stdin` and `value_iterator` are sequential streams processed lazily.
- **Section 4.6**: MapReduce rides on message-based coordination among machines; Hadoop's file system distributes the data.
- **Section 4.8**: MapReduce avoids shared state entirely — the contrast case for the synchronization problems of parallel computing.
- **Chapter 1 (pure functions)**: purity licenses arbitrary ordering and parallelism.
