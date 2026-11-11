# Section 2.2: Data Abstraction

## Core Idea
Isolate the parts of a program that use compound data from the parts that define how that data is represented, connecting them through a small set of constructor and selector functions. This methodology — **data abstraction** — makes programs easier to design, maintain, and modify.

## Frameworks Introduced
- **Data abstraction**: the general technique of isolating how data are represented from how data are manipulated.
  - When to use: whenever a program handles compound values (positions, rationals, records).
  - How: (1) define operations purely in terms of a constructor and selectors; (2) define the concrete representation separately; (3) let only the constructor/selectors touch the representation.
- **Wishful thinking**: start programming against functions you haven't implemented yet.
  - When to use: at design time, before committing to a representation.
  - How: assume `rational(n, d)`, `numer(x)`, `denom(x)` exist; write `add_rationals`, `mul_rationals` etc. on top; implement the three functions last.
- **Abstraction barriers**: layered sets of operations where each layer calls only the layer directly below.
  - When to use: structuring any multi-layer data program and judging where a change will ripple.
  - How: enumerate the layers (whole values → constructor/selectors → representation); a **violation** occurs whenever code that could use a higher-level function reaches into a lower level instead. Fewer functions depending on a representation = fewer changes when it changes.
- **Behavior conditions (the properties of data)**: abstract data = selectors + constructors + conditions they must satisfy.
  - When to use: to decide whether any implementation is a "valid" representation.
  - How: state the condition (e.g. if `x = rational(n, d)` then `numer(x)/denom(x) == n/d`); any implementation meeting it is interchangeable.

## Key Concepts
- **Compound data**: a value manipulable as a single unit yet having parts considered individually (e.g. lat/long pair).
- **Constructor**: function that builds a compound value (`rational(n, d)`).
- **Selector**: function that extracts a part (`numer(x)`, `denom(x)`).
- **Pair**: any way of gluing two values into one; two-element lists are one implementation, not the definition.
- **Element selection**: `pair[0]`, `pair[1]` — 0-indexed; the index is the offset from the beginning. Functional equivalent: `operator.getitem`.
- **Abstraction barrier violation**: bypassing the proper layer's operations to use a lower level directly.
- **Functional pair**: a pair implemented purely with a higher-order function — proof that functions suffice to represent compound data.

## Mental Models
- Think of data abstraction as the data analogue of functional abstraction: it separates how a value is used from how it is constructed.
- Use the three-layer table as a design checklist: who treats rationals as whole values, who as numerator/denominator, who as two-element lists — each layer uses only the column below.
- Think of "what data is" behaviorally: a pair is anything satisfying "if `p` was constructed from `x` and `y`, `select(p,0)` is `x` and `select(p,1)` is `y`" — representation is irrelevant if behavior holds.
- When improving a representation (e.g. reducing to lowest terms), change only the constructor; nothing above the barrier should need edits.

## Anti-patterns
- **Violating abstraction barriers**: `square_rational_violating_twice` (indexing the list directly) must be rewritten whenever the representation changes; `square_rational_violating_once` (using numer/denom instead of mul_rational) breaks when constructor/selector signatures change. Only the barrier-respecting version survives representation changes untouched.
- **Scattering representation knowledge**: the more functions that assume "a rational is a two-element list", the more expensive every representation change becomes.

## Code Examples
```python
def add_rationals(x, y):
    nx, dx = numer(x), denom(x)
    ny, dy = numer(y), denom(y)
    return rational(nx * dy + ny * dx, dx * dy)

def rational(n, d):        # representation: reduced pair
    g = gcd(n, d)
    return (n//g, d//g)

def numer(x):
    return x[0]

def denom(x):
    return x[1]
```
- **What it demonstrates**: arithmetic written against the abstraction; adding gcd-reduction required changing only the constructor.

## Reference Tables
| Parts of the program that... | Treat rationals as... | Using only... |
|---|---|---|
| Use rational numbers to perform computation | whole data values | add_rational, mul_rational, rationals_are_equal, print_rational |
| Create rationals or implement rational operations | numerators and denominators | rational, numer, denom |
| Implement selectors and constructor for rationals | two-element lists | list literals and element selection |

## Worked Example
Rational numbers end to end. Division of ints loses exactness (`1/3` is an approximation), so keep numerator and denominator as an exact pair. By wishful thinking, write `add_rationals`, `mul_rationals`, `print_rational`, `rationals_are_equal` against assumed `rational/numer/denom`. Implement them with a two-element list. First run: `add_rationals(third, third)` prints `6 / 9` — not reduced. Fix by dividing both parts by `gcd(n, d)` inside the constructor only; now it prints `2 / 3`, with zero changes to the arithmetic layer. Finally, show pairs themselves need no list: `pair(x, y)` returning a closure `get(index)` plus `select(p, i) = p(i)` satisfies the pair behavior condition — functions are sufficient to represent compound data.

## Key Takeaways
1. Structure data programs as layers connected by constructors and selectors; each layer uses only the operations of the layer below.
2. Use wishful thinking: write the operations first, pick the representation last.
3. Every improvement to a representation should be localized — ideally a constructor-only change.
4. Judge a representation by behavior conditions, not by its concrete type; any implementation meeting the conditions is valid and swappable.
5. Count abstraction-barrier violations when reviewing code — each one is a future maintenance cost.
6. Functions alone can represent compound data (functional pairs), so "data" is defined by behavior, not by storage.

## Connects To
- **Section 2.1**: float approximation is the problem the exact rational representation solves.
- **Section 2.3**: sequences generalize pairs; the sequence abstraction is another selector/constructor interface.
- **Section 2.4**: dispatch dictionaries and message passing extend the "data as behavior" insight to mutable data.
