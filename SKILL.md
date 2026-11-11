---
name: composing-programs
description: "Knowledge base from \"Composing Programs\" by John DeNero (UC Berkeley CS61A, Python-based SICP). Use when applying its frameworks for functional abstraction, data abstraction, higher-order functions, recursion, OOP design, interpreters, generators/lazy evaluation, or declarative/logic programming — studying the book or referencing its concepts."
---

<!-- argument-hint: [topic, framework name, or section number like ch16] -->

# Composing Programs
**Author**: John DeNero | **Sections**: 29 (4 chapters) | **Generated**: 2026-11-11 | **License**: CC BY-SA 3.0

## How to Use This Skill

- **Without arguments** — load core frameworks below
- **With a topic** — ask about `recursion`, `abstraction barriers`, `generators`; I find and read the relevant section file
- **With section** — ask for `ch16`; I load chapters/ch16-higher-order-functions.md
- **Browse** — ask "what chapters do you have?"

When a topic isn't in Core Frameworks below, read the relevant chapter file before answering.

---

## Core Frameworks & Mental Models

**Abstraction is the central move.** Every chapter is one mechanism for drawing a boundary you can think behind:
- **Functional abstraction** — a function's *what* (domain, range, intent) separated from its *how*. If you must read a function's body to use it, the abstraction failed.
- **Data abstraction** — programs that use data should make **no assumptions** about representation; separate use from representation with constructors/selectors. **Abstraction barriers**: each level uses only the interface of the level below. Violating a barrier isn't an error that crashes — it's a dependency that makes change expensive later.
- **Metalinguistic abstraction** — when a problem domain outgrows the host language, define a new language (interpreter); an interpreter is just `eval`/`apply` in mutual recursion, and it is *within reach* — Calculator fits in a page of Python.

**The environment model (not substitution) is how Python actually works.** Names mean nothing without an environment. An environment = sequence of frames. Evaluation rules:
1. Call expression: evaluate operator, evaluate operands, apply.
2. User-defined function applied → new frame whose parent is the frame **where the function was defined** (lexical scope), not where it's called.
3. Name lookup: earliest frame in the chain where the name is bound.
This single model explains closures, `nonlocal`, higher-order functions, and why mutable default args bite.

**Higher-order functions are the compression tool.** When two functions share a pattern (summation over terms, iterative improvement), extract the pattern as a function taking functions. Named patterns from the book: **iterative improvement** (`improve(update, close, guess)`), **function composition**, **curry**, **decorator**. Use a HOF when you'd otherwise copy a loop skeleton.

**Recursion discipline:** (1) base case first; (2) assume the recursive call is correct for the smaller input — **the recursive leap of faith** — and verify the combination step only; (3) tree recursion for choice problems (counting partitions: with/without the largest part). Don't trace the whole call tree in your head; verify one level.

**State and identity.** Mutable data introduces *time*: expressions stop being referentially transparent; **sameness ≠ equality** (`is` vs `==`). Prefer non-local state via closures or objects only where identity genuinely matters; keep pure functions pure (the book separates **pure vs non-pure functions** from page one).

**OOP as message passing + dispatch.** A class is a template for state plus behavior; a dispatch dictionary is the mechanism beneath the syntax (classes can be *implemented* with functions — ch26). Rules of thumb: **inherit for "is-a", compose for "has-a"**; class attributes for shared, instance attributes for per-object; use `__repr__` for programmers / `__str__` for humans; generic functions via **type dispatch, coercion, or interfaces** (shared messages like `__len__`, `__getitem__`).

**Efficiency = orders of growth, then memoize.** Measure space as *active* environments, not total calls. **Memoization** turns tree-recursive Θ(φⁿ) fib into Θ(n) — reach for it whenever a pure function is recomputed on repeated arguments.

**Lazy/implicit sequences.** An **iterator** is one-pass state over a sequence; a **generator** (`yield`) is the cheapest way to write one. Compute values on demand when sequences are long, infinite, or mostly unused. A `Stream` = explicit lazy linked list.

**Declarative vs imperative.** SQL and logic programming state *what*, the interpreter decides *how*. **Unification** = two-way pattern matching with variables on both sides. Reach for declarative tools when the problem is relations/queries, not procedures.

**Parallelism correctness tools:** locks, semaphores, conditions — but the book's deeper rule: shared mutable state is the enemy; prefer map-reduce-style functional decomposition where possible.

---

## Chapter Index

| # | Title | Key Frameworks |
|---|-------|----------------|
| [ch11](chapters/ch11-getting-started.md) | Getting Started | interactive sessions, errors as diagnostics |
| [ch12](chapters/ch12-elements-of-programming.md) | Elements of Programming | call expressions, pure/non-pure functions, environments |
| [ch13](chapters/ch13-defining-new-functions.md) | Defining New Functions | def, environment diagrams, functions as abstractions |
| [ch14](chapters/ch14-designing-functions.md) | Designing Functions | docstrings, DRY, default args |
| [ch15](chapters/ch15-control.md) | Control | statements vs expressions, conditionals, iteration, testing |
| [ch16](chapters/ch16-higher-order-functions.md) | Higher-Order Functions | HOFs, nested defs, closures, lambda, currying, decorators |
| [ch17](chapters/ch17-recursive-functions.md) | Recursive Functions | base cases, leap of faith, mutual/tree recursion |
| [ch21](chapters/ch21-introduction.md) | Data: Introduction | native types, objects |
| [ch22](chapters/ch22-data-abstraction.md) | Data Abstraction | constructors/selectors, abstraction barriers |
| [ch23](chapters/ch23-sequences.md) | Sequences | list/str/tree, slicing, comprehensions, linked lists |
| [ch24](chapters/ch24-mutable-data.md) | Mutable Data | identity vs equality, nonlocal, dispatch dictionaries |
| [ch25](chapters/ch25-object-oriented-programming.md) | Object-Oriented Programming | classes, inheritance vs composition |
| [ch26](chapters/ch26-implementing-classes-and-objects.md) | Implementing Classes & Objects | OOP built from functions |
| [ch27](chapters/ch27-object-abstraction.md) | Object Abstraction | generic functions, type dispatch, coercion, special methods |
| [ch28](chapters/ch28-efficiency.md) | Efficiency | orders of growth, memoization, space of environments |
| [ch29](chapters/ch29-recursive-objects.md) | Recursive Objects | Link/Tree classes, sets as ordered structures |
| [ch31](chapters/ch31-introduction.md) | Interpreters: Introduction | metalinguistic abstraction |
| [ch32](chapters/ch32-functional-programming.md) | Functional Programming (Scheme) | scheme basics, tail recursion |
| [ch33](chapters/ch33-exceptions.md) | Exceptions | raise, try/except as control flow |
| [ch34](chapters/ch34-interpreters-for-languages-with-combination.md) | Interpreters: Combination | Calculator, eval/apply, REPL |
| [ch35](chapters/ch35-interpreters-for-languages-with-abstraction.md) | Interpreters: Abstraction | full Scheme interpreter, data as programs |
| [ch41](chapters/ch41-introduction.md) | Data Processing: Introduction | pipelines |
| [ch42](chapters/ch42-implicit-sequences.md) | Implicit Sequences | iterators, generators, yield, streams, lazy evaluation |
| [ch43](chapters/ch43-declarative-programming.md) | Declarative Programming | SQL select/join, aggregation |
| [ch44](chapters/ch44-logic-programming.md) | Logic Programming | facts, queries, Prolog-style rules |
| [ch45](chapters/ch45-unification.md) | Unification | two-way pattern matching, search |
| [ch46](chapters/ch46-distributed-computing.md) | Distributed Computing | client/server, message protocols |
| [ch47](chapters/ch47-distributed-data-processing.md) | Distributed Data Processing | map-reduce |
| [ch48](chapters/ch48-parallel-computing.md) | Parallel Computing | threads, locks, synchronization pitfalls |

## Topic Index

- **Abstraction barriers** → ch22
- **Closures / nested defs** → ch16
- **Comprehensions** → ch23
- **Decorators** → ch16
- **Dispatch dictionaries** → ch24, ch26
- **Environment model / diagrams** → ch12, ch13, ch16
- **Exceptions** → ch33
- **Generators / yield / iterators** → ch42
- **Generic functions / coercion** → ch27
- **Higher-order functions / currying / lambda** → ch16
- **Inheritance vs composition** → ch25
- **Interpreters / eval / apply / REPL** → ch34, ch35
- **Linked lists** → ch23, ch29
- **Locks / threads / synchronization** → ch48
- **Memoization / orders of growth** → ch28
- **Mutability / identity vs equality / nonlocal** → ch24
- **Pure vs non-pure functions** → ch12
- **Recursion / tree recursion / base cases** → ch17
- **Scheme / tail recursion** → ch32
- **Sequences / slicing / trees** → ch23
- **SQL / joins / aggregation** → ch43
- **Streams / lazy evaluation** → ch42
- **Unification / logic programming** → ch44, ch45

## Supporting Files

- [glossary.md](glossary.md) — key terms with definitions
- [patterns.md](patterns.md) — techniques and design patterns
- [cheatsheet.md](cheatsheet.md) — decision rules and quick reference

---

## Scope & Limits

Covers the book content only (2nd edition, composingprograms.com, CC BY-SA). Source text is openly licensed; chapter files are synthesized summaries, not raw text. For hands-on work in a codebase, combine with project-specific tools.
