# Section 4.1: Introduction

## Core Idea
Large data sets are processed efficiently by organizing programs as pipelines of manipulations over sequential streams of data; the sequence concept from Chapter 2 extends to collections of unbounded or infinite size.

## Frameworks Introduced
- **Pipelines over sequential streams**: structure data processing as a chain of transformations applied to a stream, rather than loading and mutating a complete in-memory collection.
  - When to use: data sets too large to materialize, or sequences that grow without bound (sensor feeds, event logs, user activity).
  - How: represent data as a sequence interface; compose stage after stage of manipulation; let each stage pull elements as needed.

## Key Concepts
- **Sequential data stream**: a series of elements consumed in order, possibly unbounded.
- **Unbounded sequence**: a collection with no fixed size that grows as the world evolves (phone calls through a tower, mouse movements, sensor readings).
- **Infinite sequence**: a mathematically endless series such as the positive integers or Fibonacci numbers.
- **Sequence interface**: the Chapter 2 abstraction (`list`, `range`) that this chapter generalizes beyond stored, finite collections.

## Mental Models
- Think of a data set as a stream flowing through a pipeline of stages, not as a block of memory you own.
- Use unbounded-sequence thinking when the data source outlives any single program run.

## Anti-patterns
- **Materializing everything up front**: fails on unbounded or very large data; the point of the chapter is computing elements on demand instead.

## Code Examples
(No code in this introductory section; the chapter's techniques start in 4.2.)

## Worked Example
The author motivates the chapter with real-world unbounded sequences: telephone calls through a cell tower, a user's mouse movements, accelerometer readings from an aircraft — all keep growing, so a program processing them must treat them as implicit, lazily produced sequences rather than stored lists.

## Key Takeaways
1. Extend the sequence abstraction to unbounded and infinite collections.
2. Organize big-data programs as pipelines over sequential streams.
3. Compute elements on demand instead of storing them all.

## Connects To
- **Section 4.2**: implements this idea concretely via iterators, generators, and streams.
- **Section 4.7**: scales the pipeline idea to distributed MapReduce processing.
