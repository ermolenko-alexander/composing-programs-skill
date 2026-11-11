# Section 2.1: Introduction

## Core Idea
Chapter 1 concentrated on computational processes and the role of functions in program design: primitive data (numbers), primitive operations (arithmetic), compound functions formed through composition and control, functional abstractions created by naming processes, and higher-order functions that let us manipulate — and thereby reason in terms of — general methods of computation. DeNero calls this "much of the essence of programming."

Chapter 2 focuses on **data**. The techniques here allow us to represent and manipulate information about many different domains; with the explosive growth of the Internet, vast amounts of structured information are freely available, and computation can be applied to a vast range of problems. Effective use of built-in and user-defined data types is fundamental to data processing applications.

The ground layer: every value in Python has a *class* that determines what type of value it is, and values that share a class share behavior. Native types are the small, memorizable set of primitives from which all user-defined data abstractions are built.

## Frameworks Introduced
- **Native data types**: the types built into the Python language, defined by two properties (see Evaluation Rules below).
  - When to use: as the foundation layer — everything above them is built with means of combination and abstraction developed in this chapter.
- **int vs float distinction**: ints are exact and unbounded; floats are finite-precision approximations with minimum and maximum values.
  - When to use: whenever equality tests or precision matter.
  - How: treat every `float` as an approximation to a real value; never rely on float equality; remember `int / int` yields a `float`.
- **Class as behavior determinant**: inspecting `type(x)` tells you what operations a value supports and how it will behave.

## Evaluation Rules
The author's formal statements in this section:

1. **Class rule**: Every value in Python has a class that determines what type of value it is. Values that share a class also share behavior (e.g. the integers `1` and `2` are both instances of the `int` class; both can be negated or added to another integer).
2. **type() rule**: The built-in `type` function inspects the class of any value.

   ```python
   >>> type(2)
   <class 'int'>
   ```
3. **Definition of native data types** — a type is native when it has both properties:
   - There are expressions that evaluate to values of native types, called **literals**.
   - There are built-in functions and operators to manipulate values of native types.

   Example: the `int` class is the native type for integers. Integer literals (sequences of adjacent numerals) evaluate to `int` values, and mathematical operators manipulate these values.
4. **int exactness rule**: `int` objects represent integers exactly, without any approximation or limits on their size.
5. **float approximation rule**: `float` objects can represent a wide range of fractional numbers, but not all numbers can be represented exactly, and there are minimum and maximum values. Therefore float values should be treated as approximations to real values, with only a finite amount of precision. Combining float values can lead to approximation errors.
6. **True division rule**: dividing one `int` by another yields a `float` value — a truncated finite approximation to the actual ratio of the two integers divided.

   ```python
   >>> type(1/3)
   <class 'float'>
   >>> 1/3
   0.3333333333333333
   ```
7. **Consistency across languages**: these int/float details must be memorized by programmers, but there are only a handful of native data types, and the same details are consistent across many programming languages, enforced by community guidelines such as the IEEE 754 floating point standard.

## Native Numeric Types (Reference Table)
Python includes exactly three native numeric types:

| Class | Represents | Literal example | Key property |
|---|---|---|---|
| `int` | integers | `12` | Exact; no approximation or size limits |
| `float` | real numbers | `1.5` | Floating-point approximation; finite precision; min/max values exist |
| `complex` | complex numbers | `1+1j` | Third native numeric type |

```python
>>> type(1.5)
<class 'float'>
>>> type(1+1j)
<class 'complex'>
```

The name `float` comes from how real numbers are represented in Python and many other languages: a "floating point" representation (the representation details are not a topic for the text, but the high-level int/float differences are important to know).

## Key Concepts
- **Class**: what every Python value has; it determines the value's type, and values sharing a class share behavior.
- **`type` function**: built-in for inspecting the class of any value — the first diagnostic.
- **Native data type**: a type built into the language, defined by having literals plus built-in functions/operators.
- **Literal**: an expression that evaluates directly to a value of a native type (e.g. `12`, `1.5`, `1+1j`).
- **int**: exact integer representation with no size limit.
- **float**: floating-point approximation of real numbers; finite precision; min and max values; governed by the IEEE 754 standard.
- **complex**: Python's third native numeric type.
- **True division (`/`)**: on two ints returns a float — a truncated finite approximation of the ratio, never an exact rational.
- **bool**: the native class for `True` and `False` — the section's example of a non-numeric native type.
- **Approximation error**: the accumulated rounding that makes mathematically equal float expressions produce different results.

## Worked Examples

### 1. Exact unbounded integers
```python
>>> 12 + 3000000000000000000000000
3000000000000000000000012
```
Walkthrough: both operands are integer literals, so both evaluate to `int` values. The `+` operator is a built-in operator on the native `int` type. Because ints are exact with no size limit, the result is the precise sum — no overflow, no rounding — even though the second operand far exceeds any machine word size. This exemplifies both defining properties of a native type at once (literals + built-in operators) and the int exactness rule.

### 2. Float approximation: two expressions that "should" both be 7
Mathematically, `7/3*3` and `1/3*7*3` are both exactly 7. In Python:
```python
>>> 7 / 3 * 3
7.0
>>> 1 / 3 * 7 * 3
6.999999999999999
```
Walkthrough: in the first expression, `7 / 3` produces the float approximation `2.3333…`, and multiplying by 3 happens to round back to exactly `7.0`. In the second, `1 / 3` produces `0.3333333333333333` — already an approximation; multiplying by 7 and then 3 compounds the rounding error, landing at `6.999999999999999`. The same mathematical computation, reordered, yields different answers because each intermediate float carries only finite precision. As the author puts it, both expressions would evaluate to 7 *if not for approximation*. This is the chapter's motivating case for building an exact compound representation (the rational numbers of Section 2.2) instead of relying on native division.

### 3. The equality-test trap
```python
>>> 1/3 == 0.333333333333333312345  # Beware of float approximation
True
```
Walkthrough: `1/3` evaluates to the float `0.3333333333333333`. The literal on the right, `0.333333333333333312345`, has more digits than a float can store, so it too is rounded to the nearest representable float — and after rounding, both sides are the *same* float. The comparison returns `True` even though the written numbers are obviously unequal as real numbers. Problems with float approximation appear precisely when we conduct equality tests: the comparison is exact, but both operands were silently approximated first.

## Common Errors
Mistakes the author explicitly warns about in this section:

1. **Trusting float equality**: `1/3 == 0.333333333333333312345` is `True`. Float comparison silently accepts approximation error because both operands are rounded to representable floats before comparing. Never test floats for exact equality when exactness matters.
2. **Expecting float arithmetic to match mathematics**: `1 / 3 * 7 * 3` gives `6.999999999999999`, not `7`. Combining float values can lead to approximation errors even when every operand started as an exact int.
3. **Assuming `/` between ints is exact**: `int / int` yields a float — a truncated finite approximation of the true ratio. If you need the exact ratio, you must keep numerator and denominator as separate ints (which is exactly what Section 2.2's rational-number abstraction does).
4. **Forgetting float limits**: floats have minimum and maximum representable values, unlike ints, which have no size limits at all.

## Non-Numeric Types
Values can represent many other kinds of data: sounds, images, locations, web addresses, network connections, and more. A few are represented by native types — the `bool` class for `True` and `False` is the example given. But the type for *most* values must be defined by programmers, using the means of combination and abstraction developed in the rest of Chapter 2. (For exhaustive practical coverage of all native types, the author points to the "native data types" chapter of *Dive Into Python 3*.)

## Mental Models
- A float is an approximation carrying rounding error, never the real number itself. `7 / 3 * 3` is `7.0` only by luck of how the rounding falls; `1 / 3 * 7 * 3` is `6.999999999999999`.
- `type(x)` is the first diagnostic when a value behaves unexpectedly — behavior follows class.
- Native types are the small memorized vocabulary of a language. The memorization burden is deliberately light: only a handful of types, with details consistent across languages via standards like IEEE 754. Everything else — sounds, images, positions, connections — must be defined by programmers through combination and abstraction.

## Anti-patterns
- **Equality tests on floats**: exact comparison of approximated values silently accepts rounding error (`True` for visibly different decimals).
- **Assuming division preserves exactness**: dividing one int by another yields a truncated float approximation of the ratio; keep numerator and denominator separately when exactness matters (motivates 2.2's rational numbers).
- **Treating int/float as interchangeable**: the subtle class differences "have wide-ranging consequences for writing programs" — they are details that must be memorized, not papered over.

## Complete Code Listings (as in the book)
```python
>>> type(2)
<class 'int'>

>>> 12 + 3000000000000000000000000
3000000000000000000000012

>>> type(1.5)
<class 'float'>
>>> type(1+1j)
<class 'complex'>

>>> 7 / 3 * 3
7.0
>>> 1 / 3 * 7 * 3
6.999999999999999

>>> type(1/3)
<class 'float'>
>>> 1/3
0.3333333333333333

>>> 1/3 == 0.333333333333333312345  # Beware of float approximation
True
```

## Key Takeaways
1. Every value has a class; values sharing a class share behavior. Inspect with `type(x)`.
2. A type is "native" iff it has literals plus built-in operators/functions that manipulate its values.
3. Python has exactly three native numeric types: `int`, `float`, `complex`.
4. ints are exact and unbounded; floats are bounded, finite-precision approximations — never test floats for exact equality, and expect reordered arithmetic to give different results.
5. `/` always produces a float, even between two ints — a truncated approximation of the true ratio.
6. Float behavior is standardized (IEEE 754) and consistent across languages, so the handful of details is memorized once.
7. Native types are few; the type for most real-world data (sounds, images, locations, connections) must be constructed by the programmer — the subject of the rest of Chapter 2.

## Connects To
- **Chapter 1**: this section recaps its arc — primitive data and operations, composition and control, functional abstraction by naming, higher-order functions as general methods.
- **Section 2.2 (Data Abstraction)**: float approximation of ratios directly motivates the rational-number data abstraction built from exact ints kept as separate numerator and denominator.
- **Section 2.3 (Sequences)**: the following sections introduce more native types (lists, strings) focusing on their role in creating useful data abstractions.
