# Section 4.7: Distributed Data Processing

## Core Idea
When a dataset is too large for one machine, distribute it across many and coordinate with MapReduce: express the application as pure map and reduce functions, and let the framework own all communication, coordination, and failure handling.

## Frameworks Introduced
- **MapReduce**: computation over a large, unordered stream of input values in three steps:
  1. A map function is applied to each input, emitting zero or more intermediate key-value pairs.
  2. All intermediate pairs are grouped by key (the sort phase makes same-key pairs adjacent).
  3. A reduce function combines the values for each key, emitting zero or more output values per key.
  - When to use: batch processing over data distributed among many machines where per-machine results must be aggregated.
  - How: write only the two pure functions; the framework creates map tasks and reduce tasks, runs mappers in parallel (map phase) and reducers in parallel (reduce phase), and handles all inter-task communication and grouping.
- **Separation of concerns**: (1) map/reduce functions that process data vs (2) communication and coordination between machines — machine failures, network failures, progress monitoring are hidden from the application developer by abstraction.
- **Pure functions enable parallelism**: MapReduce requires map and reduce to be pure; a program of pure functions can be computed in arbitrary order and in parallel without changing the result.
- **Unix pipes as a local MapReduce**: `cat input | ./mapper.py | sort | ./reducer.py` — standard input (`sys.stdin`, iterable of lines), standard output (`print`), and the pipe `|` compose Unix programs; `sort` plays the sort phase. "The power of a system comes more from the relationships among programs than from the programs themselves" (Kernighan and Pike).
- **Hadoop (distributed implementation)**: open-source MapReduce on a cluster; its streaming interface lets arbitrary Unix programs serve as map/reduce functions — the same mapper/reducer files run unchanged. Advantages over the local version: speed (parallel tasks on many machines), fault tolerance (failed tasks recomputed elsewhere), monitoring (progress UI).

## Key Concepts
- **Map task**: applies the map function to a subset of the input, outputs intermediate key-value pairs.
- **Reduce task**: sorts/groups pairs by key, applies the reduce function per key.
- **Map / sort / reduce phases**: parallel mappers, grouping by sorted key, parallel reducers.
- **emit**: outputs a key-value pair (from the companion `mr` module).
- **values_by_key**: groups an input stream into (key, value_iterator) pairs for reducing.
- **Standard input / standard output**: the iterable input and line-oriented output that make a Python file a composable Unix program (`#!/usr/bin/env python3`, `chmod u+x`).
- **Streaming interface**: Hadoop's mechanism for using any Unix program as mapper or reducer.

## Mental Models
- Think of MapReduce as functional programming industrialized: purity buys the freedom to reorder and parallelize.
- Think of `mapper | sort | reducer` as the entire framework in one shell line — Hadoop is the same pipeline with parallelism, fault tolerance, and monitoring added.
- Choose keys so that everything needing to be combined shares a key: the key is the unit of aggregation.

## Anti-patterns
- **Impure map or reduce functions**: side effects or order dependence break the framework's freedom to reorder, rerun (fault tolerance re-executes tasks!), and parallelize.
- **Hand-rolling machine coordination**: failure handling, communication, and progress tracking belong to the framework; writing them yourself forfeits the abstraction.

## Code Examples
```python
#!/usr/bin/env python3
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
```python
#!/usr/bin/env python3
import sys
from mr import values_by_key, emit

for key, value_iterator in values_by_key(sys.stdin):
    emit(key, sum(value_iterator))
```
- **What it demonstrates**: a complete MapReduce application — mapper emitting (vowel, count) pairs and a reducer summing values per key.

## Worked Example
Counting vowels in a haiku. Input file `haiku.txt` ("Google MapReduce / Is a Big Data framework / For batch processing"). Map: for each line, emit (vowel, count) for each vowel with nonzero count. Reduce: built-in `sum` over each key's value iterator. Local run: `cat haiku.txt | ./count_vowels_mapper.py | sort | ./sum_reducer.py` yields a=6, e=5, i=2, o=5, u=1. The identical programs run distributed via `python3 mr.py run count_vowels_mapper.py sum_reducer.py [input] [output]` against a Hadoop file system.

## Key Takeaways
1. Express distributed data processing as pure map and reduce functions; the framework does the rest.
2. The three-step model (map → group by key → reduce) covers a huge class of batch computations.
3. Prototype locally with Unix pipes; the same mapper/reducer scale to a cluster via Hadoop streaming.
4. Purity is the contract that makes parallel execution and fault-tolerant re-execution safe.
5. Framework benefits: parallel speed, recomputation on failure, progress monitoring — all without exposing complexity.

## Connects To
- **Section 4.2**: inputs and value iterators are sequential streams processed lazily.
- **Section 4.6**: MapReduce rides on message-based coordination among machines.
- **Chapter 1 (pure functions)**: purity is what licenses arbitrary ordering and parallelism.
