# Section 1.4: Designing Functions

## Core Idea
Functions are the primary medium for expressing computational processes; having covered their formal properties and application, this section asks what makes a *good* function. "Fundamentally, the qualities of good functions all reinforce the idea that functions are abstractions." The guidelines improve readability, reduce errors, and often minimize the total amount of code written; decomposing a complex task into concise functions is a skill that takes experience to master.

## Frameworks Introduced

### The Three Guidelines for Good Functions (the author's formulation)
1. **Exactly one job.** Each function should have exactly one job. That job should be identifiable with a short name and characterizable in a single line of text. Functions that perform multiple jobs in sequence should be divided into multiple functions.
2. **Don't repeat yourself (DRY)** — "a central tenet of software engineering." Multiple fragments of code should not describe redundant logic; that logic should be implemented once, given a name, and applied multiple times. "If you find yourself copying and pasting a block of code, you have probably found an opportunity for functional abstraction."
3. **Define functions generally.** Squaring is not in the Python Library precisely because it is a special case of the `pow` function, which raises numbers to arbitrary powers.

When to use: every time you write or review a function.

### Docstring Convention
- Documentation describing the function, indented along with the function body, conventionally triple-quoted.
- **The first line describes the job of the function in one line.** Following lines can describe arguments and clarify behavior.
- `help(fn)` displays the docstring (type `q` to quit Python help).
- Include docstrings "for all but the simplest functions. Remember, code is written only once, but often read many times." Python's official docstring guidelines maintain cross-project consistency.

### Comments
Attached to the end of a line after `#` (e.g., `# Boltzmann's constant`). They never appear in `help` and are ignored by the interpreter — "they exist for humans alone."

### Default Argument Values
A consequence of defining general functions is additional arguments, and functions with many arguments are awkward to call and hard to read. Defaults resolve this: in the `def` header, `param=value` makes the argument optional at call time; if not provided, the default value is bound to the formal parameter name; if provided, the default is ignored.

**The two meanings of `=`** (explicit in the text): in the `def` statement header, `=` does *not* perform assignment — it indicates a default value to use when the function is called. By contrast, an assignment statement in the body (e.g., to `k`) is an ordinary binding.

**Guideline**: most data values used in a function's body should be expressed as default values to named arguments, so they are easy to inspect and can be changed by the caller. Values that never change — like the fundamental constant `k` — can be bound in the function body or in the global frame.

## Key Concepts
- **Docstring**: embedded, help-visible documentation; first line = the one-line job.
- **Comment**: `#`-prefixed, human-only, invisible to `help`.
- **Default value**: optional-argument fallback declared in the header.
- **Generality**: design for the general case (arbitrary powers, arbitrary particle counts) and specialize via arguments.
- **One-job decomposition**: sequencing of jobs inside one function is the signal to split.
- **DRY principle**: name logic once; copy-paste is the smell.
- **The two `=`s**: header default-marker vs body assignment.
- **`help()`**: the interactive window into a function's documented contract.

## Mental Models
- **The one-line test**: if you can't characterize a function's job in a single line of text, split it.
- **Copy-paste as a warning light**: duplication is the detector for a missing functional abstraction.
- **Tunable vs fixed**: ask of each literal in a body, "might a caller want to inspect or change this?" Yes → named default argument; no (physical constant) → bind in the body or global frame.
- **Docs as the abstraction's surface**: the docstring states the intent (from 1.3.6's domain/range/intent) so the body can stay a black box.

## Anti-patterns
- **Multi-job functions**: jobs performed in sequence inside one function hide structure and resist short naming.
- **Redundant logic**: repeated fragments multiply errors and code volume.
- **Over-specific functions**: special-case utilities when a general parameterized version serves more callers (the `square`-vs-`pow` lesson).
- **Burying tunable values in the body**: caller-variable data hard-coded as literals instead of named defaults.
- **Skipping documentation**: trivial functions aside, undocumented code taxes every future reader.

## Code Examples
Documentation version (three required arguments):
```python
>>> def pressure(v, t, n):
        """Compute the pressure in pascals of an ideal gas.

        Applies the ideal gas law: http://en.wikipedia.org/wiki/Ideal_gas_law

        v -- volume of gas, in cubic meters
        t -- absolute temperature in degrees kelvin
        n -- particles of gas
        """
        k = 1.38e-23  # Boltzmann's constant
        return n * k * t / v
>>> help(pressure)   # shows the docstring
```
Default-argument version:
```python
>>> def pressure(v, t, n=6.022e23):
        """Compute the pressure in pascals of an ideal gas.

        v -- volume of gas, in cubic meters
        t -- absolute temperature in degrees kelvin
        n -- particles of gas (default: one mole)
        """
        k = 1.38e-23  # Boltzmann's constant
        return n * k * t / v
>>> pressure(1, 273.15)
2269.974834
>>> pressure(1, 273.15, 3 * 6.022e23)
6809.924502
```
What it demonstrates: docstring format, an end-of-line comment, a default for the common case, and a fixed constant bound in the body.

## Worked Example: Designing `pressure` End to End
1. **Generality first**: the ideal gas law relates pressure to volume, temperature, *and* particle count, so the function takes all three — not a hard-coded one-mole special case.
2. **Document the contract**: first docstring line states the one job ("Compute the pressure in pascals of an ideal gas"); subsequent lines name each argument's meaning and units; a link clarifies the law applied.
3. **Separate the constants**: Boltzmann's constant never changes → bound to `k` in the body with a comment; particle count varies by caller but has a dominant common value → default `n=6.022e23` in the header.
4. **Call it both ways**: `pressure(1, 273.15)` takes `n` from the header default → 2269.974834; `pressure(1, 273.15, 3 * 6.022e23)` supplies a third argument, the default is ignored → 6809.924502.
The example embodies all three guidelines: one nameable job, no duplicated logic, defined generally with convenience restored by a default.

## Common Errors (what the author flags)
- Reading the header `=` as assignment — it only marks a default; the same symbol in the body is a true binding.
- Writing functions with so many required arguments they become awkward to call — the cost of generality that defaults exist to repay.
- Expecting comments to show in `help` — only docstrings do.

## Key Takeaways
1. One function, one job, one line of description; split sequenced jobs.
2. DRY: implement logic once, name it, apply it many times; copy-paste signals a missing abstraction.
3. Define functions generally; specialize via arguments and defaults (why `square` isn't in the library).
4. Write docstrings for all but trivial functions — code is written once, read many times.
5. Expose caller-variable data as named default arguments; keep never-changing constants in the body or global frame.
6. Header `=` declares a default; body `=` binds a name — two different meanings.

## Connects To
- **Section 1.3.6**: these guidelines operationalize "functions as abstractions" (domain/range/intent); the docstring is where intent is written down.
- **Section 1.5.6 (Testing)**: docstrings host doctests, closing the documentation-verification loop.
- **Section 1.6**: generality taken to its limit — parameterizing over the function itself (higher-order functions).
