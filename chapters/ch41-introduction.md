# Section 4.1: Introduction

## Core Idea
Modern computers can process vast amounts of data representing many aspects of the world. To process large data sets efficiently, programs are organized into pipelines of manipulations on sequential streams of data. This chapter extends the sequence concept from Chapter 2 — implemented in Python by built-in types such as `list` and `range` — to collections that have unbounded or even infinite size.

## Frameworks Introduced
- **Pipelines over sequential streams**: structure data processing as a chain of transformations applied to a stream, rather than loading and mutating a complete in-memory collection.
  - When to use: data sets too large to materialize, or sequences that grow without bound (sensor feeds, event logs, user activity).
  - How: represent data behind a sequence interface; compose stage after stage of manipulation; let each stage pull elements only as needed.
- **The chapter's arc** (useful as a map when applying any one section):
  1. *4.2 Implicit Sequences* — lazy computation in one process: iterators, generators, streams.
  2. *4.3 Declarative Programming* — describe the result, not the steps: SQL `select`, joins, recursion, aggregation.
  3. *4.4–4.5 Logic Programming & Unification* — queries over relations via facts, rules, and the unification algorithm.
  4. *4.6 Distributed Computing* — multiple machines coordinating via messages (client/server, peer-to-peer).
  5. *4.7 Distributed Data Processing* — MapReduce: the pipeline idea scaled across machines.
  6. *4.8 Parallel Computing* — multiple threads/processes sharing work, and the synchronization needed to keep shared state correct.

## Key Concepts
- **Sequential data stream**: a series of elements consumed in order, possibly unbounded.
- **Unbounded sequence**: a collection with no fixed size that grows as the world evolves. The author's examples: the sequence of telephone calls sent through a cell tower, the sequence of mouse movements made by a computer user, and the sequence of acceleration measurements from sensors on an aircraft — all continue to grow as the world evolves.
- **Infinite sequence**: a mathematically endless series such as the positive integers or the Fibonacci numbers.
- **Sequence interface**: the Chapter 2 abstraction that this chapter generalizes beyond stored, finite collections.
- **Big data sets**: from them "we can learn about human behavior in unprecedented ways: how language is used, what photos are taken, what topics are discussed."

## Mental Models
- Think of a data set as a stream flowing through a pipeline of stages, not as a block of memory you own.
- Use unbounded-sequence thinking when the data source outlives any single program run: the program must process elements as they arrive, because "all elements" never exists at once.
- The same pipeline shape recurs at three scales in this chapter: within one function (generators), within one database (SQL), and across a cluster (MapReduce).

## Anti-patterns
- **Materializing everything up front**: fails on unbounded or very large data; the point of the chapter is computing elements on demand instead.
- **Treating "sequence" as synonymous with "list"**: a sequence is an interface (order + element access in turn), not a storage commitment.

## Code Examples
(No code in this introductory section; the chapter's techniques start in 4.2.)

## Worked Example
The author motivates the chapter with real-world unbounded sequences: telephone calls through a cell tower, a user's mouse movements, accelerometer readings from an aircraft. Each keeps growing as long as the world runs, so a program processing them must treat them as implicit, lazily produced sequences rather than stored lists. The mathematical analogues — positive integers, Fibonacci numbers — are literally infinite, yet Section 4.2 shows they can still be represented as first-class values and transformed by ordinary higher-order functions.

## Key Takeaways
1. Extend the sequence abstraction to unbounded and infinite collections.
2. Organize big-data programs as pipelines over sequential streams.
3. Compute elements on demand instead of storing them all.
4. The chapter progresses from laziness (one process) to declarative queries (one database) to distribution and parallelism (many machines/threads) — one theme, rising scale.

## Connects To
- **Chapter 2**: the sequence interface (`list`, `range`) being generalized here.
- **Section 4.2**: implements the idea concretely via iterators, generators, and streams.
- **Sections 4.3–4.5**: declarative and logic programming — stating what result you want over data.
- **Sections 4.6–4.8**: distributing and parallelizing the pipeline across machines and threads.
