# Section 1.4: Designing Functions

## Core Idea
The qualities of good functions all reinforce the idea that functions are abstractions: each should do one nameable job, logic should never be duplicated, and definitions should be general.

## Frameworks Introduced
- **Three guidelines for good functions**:
  - When to use: every time you write or review a function.
  - How: (1) *Exactly one job* — identifiable with a short name and characterizable in a single line of text; a function performing multiple jobs in sequence should be divided; (2) *Don't repeat yourself (DRY)* — logic implemented once, given a name, applied multiple times; copy-pasting a block of code signals an opportunity for functional abstraction; (3) *Define generally* — squaring isn't in the Python Library because it's a special case of `pow`.
- **Docstring convention**: triple-quoted documentation indented with the body; the first line describes the function's job in one line; following lines describe arguments and clarify behavior; shown by `help(fn)`.
- **Default argument values**: in the `def` header, `= value` after a parameter makes the argument optional; if omitted at call time, the default is bound to the formal parameter.

## Key Concepts
- **Docstring**: the function's embedded documentation; include one for all but the simplest functions — code is written once but read many times.
- **Comment (`#`)**: end-of-line text for humans alone; ignored by the interpreter and never shown in `help`.
- **Default value**: the `=` in a `def` header does *not* perform assignment — it indicates a fallback value; the same symbol in the body is ordinary binding.
- **Generality**: designing for the general case (arbitrary powers, arbitrary particle counts) instead of hard-coding the common one.

## Mental Models
- Use the one-line test: if you can't characterize a function's job in a single line of text, split it.
- Think of copy-paste as a compiler warning for missing abstraction.
- Use defaults for data values in a function's body that a caller might want to inspect or change; keep truly fixed constants (like Boltzmann's `k`) bound in the body or global frame.

## Anti-patterns
- **Multi-job functions**: jobs performed in sequence inside one function hide structure and resist naming.
- **Redundant logic**: repeated fragments multiply errors and code volume.
- **Over-specific functions**: special-case utilities when a general parameterized version serves more callers.
- **Burying tunable values in the body**: data a caller might vary should be a named default argument, not a hard-coded literal.

## Code Examples
```python
>>> def pressure(v, t, n=6.022e23):
        """Compute the pressure in pascals of an ideal gas.

        v -- volume of gas, in cubic meters
        t -- absolute temperature in degrees kelvin
        n -- particles of gas (default: one mole)
        """
        k = 1.38e-23  # Boltzmann's constant
        return n * k * t / v
```
- **What it demonstrates**: docstring format, a comment, a default argument for the common case, and a fixed constant bound in the body.

## Worked Example
`pressure` applies the ideal gas law. Defined generally, it takes volume, temperature, and particle count — but since an application commonly computes pressure for one mole, `n=6.022e23` is the default: `pressure(1, 273.15)` → 2269.97 uses it; `pressure(1, 273.15, 3 * 6.022e23)` → 6809.92 overrides it. Boltzmann's constant `k`, which never changes, stays in the body.

## Key Takeaways
1. One function, one job, one line of description.
2. DRY: name repeated logic once instead of copying it.
3. Define functions generally; specialize via arguments and defaults.
4. Write docstrings for all but trivial functions — first line states the job.
5. Expose caller-variable data as default arguments; keep immutable constants in the body or global frame.

## Connects To
- **Section 1.3.6**: these guidelines operationalize "functions as abstractions" (domain/range/intent).
- **Section 1.5.6 (Testing)**: docstrings host doctests, closing the documentation-verification loop.
- **Section 1.6**: generality taken further — parameterizing over the function itself.
