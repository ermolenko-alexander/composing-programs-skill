# Section 2.1: Introduction

## Core Idea
Chapter 1 was about processes and functions; Chapter 2 is about data. Every Python value has a class that determines its type, and values sharing a class share behavior — native types are the primitives from which all user-defined data abstractions are built.

## Frameworks Introduced
- **Native data types**: types built into the language, defined by two properties.
  - When to use: as the foundation layer — everything above them is built with means of combination and abstraction.
  - How: a type is native when (1) there are **literals** — expressions that evaluate to values of that type — and (2) there are built-in functions and operators that manipulate those values.
- **int vs float distinction**: ints are exact and unbounded; floats are finite-precision approximations.
  - When to use: whenever equality tests or precision matter.
  - How: treat every `float` as an approximation to a real value; never rely on float equality; remember `int / int` yields a `float`.

## Key Concepts
- **Class**: what every Python value has; it determines the value's type and behavior (inspect with `type(x)`).
- **Literal**: an expression that evaluates directly to a value of a native type (e.g. `12`, `1.5`, `1+1j`).
- **int**: exact integer representation with no size limit (`12 + 3000000000000000000000000` is exact).
- **float**: floating-point approximation of real numbers; finite precision, min/max limits, governed by the IEEE 754 standard.
- **complex**: Python's third native numeric type (`1+1j`).
- **bool**: native type of `True` and `False` — an example of a non-numeric native type.
- **True division**: `/` on two ints returns a float approximation, not an exact ratio.

## Mental Models
- Think of a float as an approximation carrying rounding error, never as the real number itself: `7 / 3 * 3` is `7.0` only by luck of rounding, while `1 / 3 * 7 * 3` is `6.999999999999999`.
- Use `type(x)` as the first diagnostic when a value behaves unexpectedly — behavior follows class.
- Think of native types as the small memorized vocabulary of a language; all other types (sounds, images, positions, connections) must be defined by programmers through combination and abstraction.

## Anti-patterns
- **Equality tests on floats**: `1/3 == 0.333333333333333312345` evaluates to `True` — float comparison silently accepts approximation error.
- **Assuming division preserves exactness**: dividing one int by another yields a truncated float approximation of the ratio; keep numerator and denominator separately when exactness matters (motivates 2.2's rational numbers).

## Code Examples
```python
>>> type(2)
<class 'int'>
>>> 1/3 == 0.333333333333333312345  # Beware of float approximation
True
>>> 12 + 3000000000000000000000000  # ints are exact and unbounded
3000000000000000000000012
```
- **What it demonstrates**: class inspection, exact int arithmetic, and the float-approximation trap.

## Worked Example
The float-precision demonstration: mathematically, `7/3*3` and `1/3*7*3` both equal 7. In Python the first happens to round to `7.0` while the second gives `6.999999999999999` — the same computation reordered yields different answers because each intermediate float carries finite precision. This is the chapter's motivating case for building an exact compound representation (a numerator/denominator pair) instead of relying on native division.

## Key Takeaways
1. Inspect any value's type with `type(x)`; behavior is determined by class.
2. A type is "native" iff it has literals plus built-in operators/functions.
3. ints are exact and unbounded; floats are bounded approximations — never test floats for exact equality.
4. `/` always produces a float, even between ints; use `//` for exact integer division when the divisor is known to divide evenly.
5. Native types are few and memorizable; the type for most real-world data must be constructed by the programmer — the subject of the rest of Chapter 2.

## Connects To
- **Section 2.2**: float approximation of ratios motivates the rational-number data abstraction built from exact ints.
- **Section 2.3**: lists and strings are the native compound types used to build sequences.
