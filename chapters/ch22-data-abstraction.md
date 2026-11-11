# Section 2.2: Data Abstraction

## Core Idea
Most things we want to represent in programs have compound structure — a geographic position has a latitude and a longitude. A **compound data** value lets a program manipulate such a thing as a single conceptual unit while still being able to consider its parts individually. The general technique of isolating the parts of a program that deal with *how data are represented* from the parts that deal with *how data are manipulated* is a powerful design methodology called **data abstraction**. It makes programs much easier to design, maintain, and modify.

Data abstraction is similar in character to functional abstraction. A functional abstraction suppresses how a function is implemented, so the function can be replaced by any other with the same overall behavior; analogously, data abstraction isolates how a compound data value is *used* from the details of how it is *constructed*. The basic idea: structure programs so that they operate on **abstract data** — making as few assumptions about the data as possible — while a concrete representation is defined as an independent part of the program. The two parts are connected by a small set of functions (constructors and selectors) that implement abstract data in terms of the concrete representation.

## Frameworks Introduced
- **Data abstraction**: isolating representation from use.
  - When to use: whenever a program handles compound values (positions, rationals, records).
  - How: (1) define operations purely in terms of a constructor and selectors; (2) define the concrete representation as an independent part of the program; (3) let only the constructor/selectors touch the representation.
- **Wishful thinking**: start programming productively before some parts of the program are implemented.
  - When to use: at design time, before committing to a representation.
  - How: assume `rational(n, d)`, `numer(x)`, `denom(x)` exist; write `add_rationals`, `mul_rationals`, `print_rational`, `rationals_are_equal` on top of them; implement the three assumed functions last.
- **Abstraction barriers**: layered sets of operations, where the functions in each layer are called by a higher level and implemented using a lower level of abstraction.
  - When to use: structuring any multi-layer data program, and judging where a representation change will ripple.
  - How: enumerate the layers (see the table below). The fewer functions that depend on a particular representation, the fewer changes are required when one wants to change that representation.
- **Behavior conditions (the properties of data)**: abstract data = a collection of selectors and constructors, together with behavior conditions they must satisfy.
  - When to use: to decide whether an implementation constitutes a valid representation of a kind of data.
  - How: state the condition (for rationals, the division property below); as long as it holds, implementation details below the barrier may change and every program written against the abstraction remains correct.

## Evaluation Rules
The section's formal content, stated as the author gives it:

1. **Structure of a data-abstract program.** A program has (a) a part that operates on abstract data, making as few assumptions about the data as possible, and (b) a part that defines a concrete representation, defined independently. They are connected by a small set of functions that implement abstract data in terms of the concrete representation.
2. **Abstraction barrier.** The underlying idea is to identify a basic set of operations in terms of which all manipulations of values of some kind will be expressed, then use *only* those operations in manipulating the data. The functions in each layer enforce an abstraction barrier: they are called by a higher level and implemented using a lower level of abstraction.
3. **Abstraction barrier violation.** A violation occurs whenever a part of the program that *can* use a higher-level function instead uses a function in a lower level. Severity can be counted in barriers crossed (one vs two, in the examples below).
4. **What constitutes data (behavior conditions).** A valid representation of a rational number is not restricted to any particular implementation; it is a value returned by `rational` that can be passed to `numer` and `denom`, and the appropriate relationship must hold: if `x` was constructed from integers `n` and `d`, then `numer(x)/denom(x)` must equal `n/d`. In general, abstract data is a collection of selectors and constructors together with behavior conditions; as long as the conditions are met, the selectors and constructors constitute a valid representation of a kind of data.
5. **Pair behavior condition.** If a pair `p` was constructed from values `x` and `y`, then `select(p, 0)` returns `x`, and `select(p, 1)` returns `y`. Anything fulfilling this condition is a pair — a two-element list and a higher-order function both qualify.
6. **Element selection.** Lists in Python (and sequences in most languages) are 0-indexed: index 0 selects the first element, index 1 the second. The intuition: the index represents how far an element is offset from the beginning of the list. The functional equivalent of the `[]` operator is `operator.getitem`, also 0-indexed. A square-brackets expression directly following another expression is element selection, not a list literal.

## Key Concepts
- **Compound data**: a value a program can manipulate as a single conceptual unit, which also has parts that can be considered individually (e.g., a latitude/longitude pair). Enables modularity: computing with whole positions shields code from representation details.
- **Abstract data**: data used with as few assumptions as possible — only through its operation set.
- **Constructor**: the function that builds a compound value — `rational(n, d)` returns the rational number with numerator `n` and denominator `d`.
- **Selector**: a function that extracts a part — `numer(x)` returns the numerator, `denom(x)` the denominator of rational `x`.
- **Pair**: any way of bundling two values together into one. Two-element lists are a common method, not the definition.
- **List literal**: expressions placed within square brackets separated by commas, e.g. `[10, 20]`. Elements are accessed by multiple assignment (`x, y = pair`) or element selection (`pair[0]`).
- **Element selection / `getitem`**: 0-indexed subscripting; `from operator import getitem; getitem(pair, 0)`.
- **Abstraction barrier**: the boundary a layer's operation set enforces between levels of abstraction.
- **Abstraction barrier violation**: bypassing a higher-level function to reach a lower level directly.
- **Behavior condition**: a required relationship among constructor and selectors (e.g., the division property) that makes a representation valid.
- **Wishful thinking**: designing against not-yet-implemented functions.
- **Functional pair**: a pair implemented purely with a higher-order function — proof that functions are sufficient to represent compound data.

## Mental Models
- Data abstraction is the data analogue of functional abstraction: it separates how a value is used from how it is constructed, just as functional abstraction separates how a function is used from how it is implemented.
- Use the three-layer table as a design checklist: who treats rationals as whole values, who as numerator/denominator, who as two-element lists — each layer uses only the operations in its own row's final column.
- Think of "what data is" behaviorally: a pair is anything satisfying the pair behavior condition; a rational is anything satisfying the division property. Representation is irrelevant while behavior holds — this is what lets you switch among representations easily.
- When improving a representation (e.g., reducing to lowest terms), aim to change only the constructor; nothing above the barrier should need edits.
- Count barriers when judging a shortcut: `square_rational_violating_once` crosses one barrier, `square_rational_violating_twice` crosses two; cost of future change scales with barriers crossed.

## Worked Examples

### 1. Rational numbers end to end
**Problem.** A rational number is a ratio of integers, written `<numerator>/<denominator>`; both parts are needed to exactly characterize the value. Actually dividing integers produces a `float` approximation:

```python
>>> 1/3
0.3333333333333333
>>> 1/3 == 0.333333333333333300000  # Dividing integers yields an approximation
True
```

**Step 1 — wishful thinking.** Assume three functions exist: `rational(n, d)` (constructor), `numer(x)` and `denom(x)` (selectors). Without saying how a rational is represented, define the arithmetic:

```python
>>> def add_rationals(x, y):
        nx, dx = numer(x), denom(x)
        ny, dy = numer(y), denom(y)
        return rational(nx * dy + ny * dx, dx * dy)

>>> def mul_rationals(x, y):
        return rational(numer(x) * numer(y), denom(x) * denom(y))

>>> def print_rational(x):
        print(numer(x), '/', denom(x))

>>> def rationals_are_equal(x, y):
        return numer(x) * denom(y) == numer(y) * denom(x)
```

Note the equality test uses cross-multiplication, so it works even on unreduced rationals.

**Step 2 — pick a representation.** Represent a rational as a pair of two integers using a two-element list:

```python
>>> def rational(n, d):
        return [n, d]

>>> def numer(x):
        return x[0]

>>> def denom(x):
        return x[1]
```

**Step 3 — run it, find the flaw.**

```python
>>> half = rational(1, 2)
>>> print_rational(half)
1 / 2
>>> third = rational(1, 3)
>>> print_rational(mul_rationals(half, third))
1 / 6
>>> print_rational(add_rationals(third, third))
6 / 9
```

The implementation does not reduce rationals to lowest terms.

**Step 4 — fix at the constructor only.** With a greatest-common-denominator function (already in the Python library), reduce before constructing. Floor division `//` is exact here because `g` divides both `n` and `d` evenly:

```python
>>> from fractions import gcd
>>> def rational(n, d):
        g = gcd(n, d)
        return (n//g, d//g)

>>> print_rational(add_rationals(third, third))
2 / 3
```

The improvement was accomplished by changing the constructor without changing any of the functions that implement the arithmetic operations. (Note the book also quietly swaps the list for a tuple here — another representation change invisible above the barrier. In modern Python, `gcd` lives in `math`, not `fractions`.)

### 2. Abstraction barriers: three ways to square a rational
All three have correct behavior; only the first is robust to future changes.

```python
>>> def square_rational(x):
        return mul_rational(x, x)
```
Best: assumes nothing about the implementation of a rational number; would not require updating even if the representation changed.

```python
>>> def square_rational_violating_once(x):
        return rational(numer(x) * numer(x), denom(x) * denom(x))
```
Violates one barrier (refers directly to numerators and denominators); would need to change whenever the selector or constructor signatures changed.

```python
>>> def square_rational_violating_twice(x):
        return [x[0] * x[0], x[1] * x[1]]
```
Violates two barriers (assumes rationals are two-element lists); would require updating whenever the implementation of rational numbers changed.

### 3. Functional pairs: data from functions alone
We never said much about what a pair *was* — only that the language supplied means to create and manipulate two-element lists. The behavior we require is just that a pair glues two values together (the pair behavior condition above). So the `list` type is not needed; two functions `pair` and `select` fulfill the description just as well:

```python
>>> def pair(x, y):
        """Return a function that represents a pair."""
        def get(index):
            if index == 0:
                return x
            elif index == 1:
                return y
        return get

>>> def select(p, i):
        """Return the element at index i of pair p."""
        return p(i)

>>> p = pair(20, 14)
>>> select(p, 0)
20
>>> select(p, 1)
14
```

This use of higher-order functions corresponds to nothing like our intuitive notion of what data should be — yet these functions suffice: **functions are sufficient to represent compound data.** The point is not that Python actually works this way (lists are implemented more directly, for efficiency reasons), but that it *could*: the functional representation, although obscure, fulfills the only conditions pairs need to fulfill. The practice of data abstraction is what allows us to switch among such representations easily.

## Reference Tables
The abstraction-barrier layers for rational numbers (different parts of the program manipulate rationals using different operations):

| Parts of the program that... | Treat rationals as... | Using only... |
|---|---|---|
| Use rational numbers to perform computation | whole data values | `add_rational`, `mul_rational`, `rationals_are_equal`, `print_rational` |
| Create rationals or implement rational operations | numerators and denominators | `rational`, `numer`, `denom` |
| Implement selectors and constructor for rationals | two-element lists | list literals and element selection |

In each layer, the functions in the final column enforce an abstraction barrier: called by a higher level, implemented using a lower level of abstraction.

Two ways to access list elements:

| Method | Syntax | Example |
|---|---|---|
| Multiple assignment (unpacking) | `x, y = pair` | binds `x` to 10, `y` to 20 |
| Element selection operator | `pair[0]` | 0-indexed; offset from the beginning |
| Functional equivalent | `getitem(pair, 0)` | `from operator import getitem` |

## Common Errors
Mistakes the author explicitly warns about:

- **Assuming integer division preserves exactness.** `1/3 == 0.333333333333333300000` evaluates to `True` — dividing integers yields a float approximation and loses the exact precision of integers. Represent exact ratios as numerator/denominator pairs instead.
- **Violating one abstraction barrier**: writing an operation like squaring in terms of `rational`/`numer`/`denom` when `mul_rational` suffices. Breaks when constructor/selector signatures change.
- **Violating two abstraction barriers**: referencing list indices (`x[0]`, `x[1]`) directly in code that should treat rationals as whole values. Breaks whenever the implementation of rational numbers changes at all.
- **Scattering representation knowledge**: the fewer functions that depend on a particular representation, the fewer changes are required to change it — every extra dependent function is a future maintenance cost.
- **Confusing a list literal with element selection**: `[10, 20]` standing alone is a literal; square brackets directly following another expression select an element, not a list value.
- **Equating "pair" with "two-element list"**: lists are one common method of bundling two values; anything satisfying the pair behavior condition is a pair. Judging data by concrete type instead of behavior conditions blocks representation switches.

## Anti-patterns
- **Reaching below the barrier** when a higher-level function exists — each violation is a function that must be rewritten on the next representation change (`square_rational_violating_once`, `square_rational_violating_twice`).
- **Committing to a representation first** instead of writing operations against assumed constructors/selectors (skipping wishful thinking) — the representation's quirks leak into every operation.
- **Fixing representation flaws above the barrier** — e.g., reducing fractions inside `add_rationals` instead of inside the constructor `rational`, spreading the fix across every operation.

## Key Takeaways
1. Structure data programs as layers connected by constructors and selectors; functions in each layer are called by the level above and implemented using the level below.
2. Use wishful thinking: write operations against assumed `rational`/`numer`/`denom` first, pick the representation last.
3. Improvements to a representation should be localized — the gcd-reduction fix touched only the constructor, zero arithmetic functions.
4. Judge a representation by behavior conditions (`numer(x)/denom(x) == n/d`; the pair select condition), not by concrete type; any implementation meeting the conditions is valid and swappable, and programs written against the abstraction remain correct.
5. Count abstraction-barrier violations when reviewing code — one barrier crossed means fragility to signature changes, two means fragility to any representation change.
6. Functions alone suffice to represent compound data (functional pairs), so "data" is defined by behavior, not by storage.

## Connects To
- **Section 2.1**: the float approximation from integer division is the problem the exact rational representation solves.
- **Section 2.3**: sequences generalize pairs; the sequence abstraction is another selector/constructor interface, and lists can hold more than two elements.
- **Section 2.4**: dispatch dictionaries and message passing extend the "data as behavior" insight to mutable data.
- **Section 1.6 (higher-order functions)**: the functional pair is a direct payoff of functions as values/closures.
- **Section 2.7 (multiple representations)**: the ease of switching representations here becomes generic functions over coexisting representations.
