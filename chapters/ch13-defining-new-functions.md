# Section 1.3: Defining New Functions

## Core Idea
Python already supplies primitives (numbers, arithmetic), combination (nested function application), and a *limited* abstraction (binding names to values). **Function definitions** are a much more powerful abstraction technique: a name is bound to a *compound operation* that can then be referred to as a unit. A precise environment model of frames explains exactly how such functions evaluate — and that model is, verbatim, "a model of evaluation" that later serves as a blueprint for implementing an interpreter.

## Frameworks Introduced

### How to define a function (the author's template)
Function definitions consist of a `def` statement indicating a `<name>` and a comma-separated list of named `<formal parameters>`, then a `return` statement — the function body — specifying the `<return expression>`, an expression to be evaluated whenever the function is applied:
```
def <name>(<formal parameters>):
    return <return expression>
```
- The second line **must** be indented (most programmers use four spaces).
- The return expression is *not* evaluated right away; it is stored as part of the newly defined function and evaluated only when the function is eventually applied.
- Both `def` statements and assignment statements bind names to values, and **any existing bindings are lost** (a name may go from a zero-argument function, to a number, to a two-argument function across successive statements).

### Environment Diagrams
An environment in which an expression is evaluated consists of a sequence of **frames**, depicted as boxes. Each frame contains **bindings**, each associating a name with its value. There is a single **global frame**. Assignment and import statements add entries to the *first* frame of the current environment.
- Functions appear in diagrams as a line starting with `func`, then the function name and formal parameters; built-in functions render as `<name>(...)` because they were never explicitly defined.
- A function's name appears twice: in the frame (the **bound name**) and as part of the function itself (the **intrinsic name**). Different names may refer to the same function, but a function has only one intrinsic name. **Only the bound name is used during evaluation**; the intrinsic name plays no role in evaluation.
- When to use diagrams: whenever the meaning of a program is non-obvious — a formal parameter sharing a name with a built-in, two functions sharing names, repeated calls. (The text's diagrams come from Philip Guo's Online Python Tutor.)

### Functions as Abstractions — the three attributes
`sum_squares` relies only on the relationship `square` defines between inputs and outputs, not on *how* the square is computed — details are suppressed, to be considered later. At that level of abstraction, "any function that computes the square is equally good" (`mul(x, x)` and `mul(x, x-1) + x` are indistinguishable by return values). A user may obtain a function as a "black box"; a programmer should not need to know how it is implemented in order to use it (the Python Library has this property).

To master a **functional abstraction**, consider its three core attributes:
- **Domain** — the set of arguments it can take.
- **Range** — the set of values it can return.
- **Intent** — the relationship it computes between inputs and output, plus any side effects.

For any `square` used by `sum_squares`: domain = any single real number; range = any non-negative real number; intent = the output is the square of the input. The attributes do not specify *how* the intent is carried out; that detail is abstracted away.

### Operators as shorthand
Expressions with infix operators have their own evaluation procedures, but can often be thought of as shorthand for call expressions: `2 + 3` ↔ `add(2, 3)`. Precedence rules determine the implicit nesting: `2 + 3 * 4 + 5` ↔ `add(add(2, mul(3, 4)), 5)` → 19; parentheses override or clarify: `(2 + 3) * (4 + 5)` ↔ `mul(add(2, 3), add(4, 5))` → 45. Division comes in two forms: `/` is `truediv` — normal division, yielding a floating point value *even if the divisor evenly divides the dividend* (`8 / 4` → `2.0`); `//` is `floordiv` — rounds the result *down* to an integer (`5 // 4` → `1`, `-5 // 4` → `-2`). Idiomatic Python prefers operators over call expressions for simple math.

## Evaluation Rules (the author's formal procedures, verbatim-faithful)

**Applying a user-defined function.** As with any call expression, the interpreter first evaluates the operator and operand expressions, then applies the named function to the resulting arguments. Applying a user-defined function introduces a second, **local** frame, only accessible to that function. To apply a user-defined function to some arguments:
1. Bind the arguments to the names of the function's formal parameters in a new **local** frame.
2. Execute the body of the function in the environment that starts with this frame.

The environment in which the body is evaluated consists of two frames: first the local frame containing formal parameter bindings, then the global frame containing everything else. **Each instance of a function application has its own independent local frame.**

**Name Evaluation rule** (the precise version): *"A name evaluates to the value bound to that name in the earliest frame of the current environment in which that name is found."*

**def-statement rule.** The entire `def` statement is processed in a single step; the body of a function is not executed until the function is called (not when it is defined). A `def` statement binds a name to a user-defined function created by the definition; an import statement binds a name to a built-in function.

**Return value.** The "Return value" shown in a call's frame is *not* a name binding; it indicates the value returned by the function call that created the frame.

**Why the machinery exists**: "All of this machinery exists to ensure that names resolve to the correct values at the correct times during program execution." All three local frames in the worked example bind `x`, to different values — local frames keep these names separate.

## Key Concepts
- **Formal parameter**: the name in a definition for the thing to be operated on (`x` in `square` names "the thing to be multiplied").
- **Frame / binding / global frame / local frame**: boxes of name→value associations; one shared global frame; a fresh local frame per application, even for repeated calls of the same function.
- **Intrinsic name vs bound name**: the function's own name vs the frame name(s) referring to it; only bound names matter in evaluation.
- **Function signature**: a description of a function's formal parameters; it tracks how many arguments are allowed (`square` takes exactly `x`; more or fewer is an error; `max(...)` takes arbitrarily many).
- **Scope**: the scope of a local name is limited to the body of the user-defined function that defines it; out of scope = no longer accessible. "This scoping behavior isn't a new fact about our model; it is a consequence of the way environments work."
- **Model of evaluation**: the conceptual framework of environments, names, and functions; some mechanical details (how a binding is implemented) are unspecified, yet it precisely and correctly describes how the interpreter evaluates call expressions.
- **Two environments even in simple code**: the top-level `square(-2)` is evaluated in the global environment, while the return expression `mul(x, x)` is evaluated in the environment created by calling `square` — `x` and `mul` are bound in that environment, in *different* frames.
- **truediv / floordiv**: the functions behind `/` and `//`.

## Mental Models
- Each function call opens a fresh private workspace (local frame); `x` in `square` and `x` in `sum_squares` can never collide.
- "Earliest frame wins" resolves any shadowing question mechanically.
- A function definition is a stored promise: the body runs at call time, never at definition time.
- A used function is a black box defined only by domain, range, intent.
- Infix is sugar: when in doubt about `2 + 3 * 4 + 5`, rewrite it mentally as nested calls.

## Anti-patterns
- **Relying on parameter names from outside**: the meaning of a function must be independent of the parameter names chosen by its author — `def square(x)` and `def square(y)` provide the same behavior. This principle is *why* parameters must remain local.
- **Confusing intrinsic and bound names**: after `f = max; max = 3`, `f(2, 3, 4)` works but `max(1, 2)` raises `TypeError: 'int' object is not callable` — the name `max` is currently bound to an integer, not a function, so it cannot be the operator of a call expression.
- **Expecting the body to run at `def` time**: it doesn't.
- **Poor names** (from the Python style guide, PEP 8, "a guide for all (non-rebellious) Python programmers"): function names lowercase, underscore-separated, descriptive; they typically evoke operations applied (`print`, `add`, `square`) or the resulting quantity (`max`, `abs`, `sum`); parameter names lowercase, single words preferred, evoking the *role* of the parameter, not just the allowed kind of argument; single letters acceptable when the role is obvious, but avoid `l`, `O`, `I` (confusable with numerals). There are many exceptions even in the standard library.

## Code Examples
```python
>>> from operator import add, mul
>>> def square(x):
        return mul(x, x)
>>> square(21)
441
>>> square(add(2, 5))
49
>>> square(square(3))
81

>>> def sum_squares(x, y):
        return add(square(x), square(y))
>>> sum_squares(3, 4)
25
```
User-defined functions are used exactly like built-ins: one cannot tell from `sum_squares` whether `square` is built into the interpreter, imported, or user-defined.

Rebinding destroys prior bindings (def and `=` alike):
```python
>>> def g():
        return 1
>>> g()
1
>>> g = 2
>>> g
2
>>> def g(h, i):
        return h + i
>>> g(1, 2)
3
```
Intrinsic vs bound names:
```python
f = max
max = 3
result = f(2, 3, 4)   # 4 — f still refers to the function
max(1, 2)             # TypeError: 'int' object is not callable
```
Operators as shorthand:
```python
>>> 2 + 3 * 4 + 5                      # 19
>>> add(add(2, mul(3, 4)), 5)          # 19
>>> (2 + 3) * (4 + 5)                  # 45
>>> mul(add(2, 3), add(4, 5))          # 45
>>> 5 / 4      # 1.25
>>> 8 / 4      # 2.0  (always floating point)
>>> 5 // 4     # 1
>>> -5 // 4    # -2  (rounds DOWN)
>>> from operator import truediv, floordiv
>>> truediv(5, 4)    # 1.25
>>> floordiv(5, 4)   # 1
```

## Worked Examples

### 1. `square(-2)` step by step (the book's four-snapshot diagram)
```python
from operator import mul
def square(x):
    return mul(x, x)
square(-2)
```
1. After the import, only `mul` is bound in the global frame.
2. The `def` statement executes *in a single step*, binding `square` to a new function; the body does not run.
3. The call `square(-2)` creates a new frame binding formal parameter `x` to `-2`.
4. `mul(x, x)` is evaluated in the two-frame environment [square's local, global]: `x` → -2 (local, both occurrences), `mul` → built-in (global); the function returns 4, recorded as "Return value" (not a binding).
Two different environments were used: the top-level call evaluated in the global environment, the return expression in the call's environment.

### 2. `result = sum_squares(5, 12)` → 169 (three local frames)
```python
from operator import add, mul
def square(x):
    return mul(x, x)
def sum_squares(x, y):
    return add(square(x), square(y))
result = sum_squares(5, 12)
```
- Python evaluates the name `sum_squares` (bound to a user-defined function in the global frame) and numerals 5, 12.
- Applying it introduces a local frame binding x=5, y=12. The body is the call expression with operator `add` and operands `square(x)`, `square(y)` — all three subexpressions evaluated in the environment beginning with the `sum_squares` frame.
- `add` is found in the global frame (built-in addition). Operand 0: `square` names a function in the global frame, `x` names 5 in the local frame; applying `square` to 5 introduces *yet another* local frame binding x=5, where `mul(x, x)` → 25.
- Operand 1: `y` names 12; `square`'s body runs again in a *third* local frame binding x=12 → 144.
- `add(25, 144)` → 169, the return value of `sum_squares`.
All three local frames bind `x` to different values; local frames keep the names separate — this example is why the model requires the complexity introduced.

### 3. Shadowing resolved by the Name Evaluation rule
In `sum_squares`'s body, `x` is found in the earliest frame (local, =5), never confused with any other `x`. In `square`'s body during operand 0, the current environment is [square's frame, global] — note that `sum_squares`'s frame is *not* in it; `x` resolves to 5 from square's own frame. If parameters were not local, `x` in `square` could be confused with `x` in `sum_squares`; critically, bindings for `x` in different local frames are unrelated by design.

## Common Errors (what the author explicitly flags)
- Forgetting the mandatory indentation of the function body.
- Supplying more or fewer arguments than a function's signature allows — an error.
- `TypeError: 'int' object is not callable` — calling a name whose binding was clobbered (bound names, not intrinsic names, drive evaluation).
- Assuming `def` runs the body; it only stores it.
- Assuming `8 / 4` yields an integer — `/` always yields floating point; and assuming `//` truncates toward zero — it rounds *down* (`-5 // 4 == -2`).
- Choosing confusable parameter names (`l`, `O`, `I`).

## Key Takeaways
1. Apply user-defined functions by the two-step rule: bind arguments in a new local frame; execute the body in the environment starting with that frame ([local, global]).
2. Resolve any name by the Name Evaluation rule: earliest frame of the current environment in which the name is found.
3. Every call gets an independent local frame; parameter names are local and interchangeable — the meaning of a function is independent of its author's parameter names.
4. Characterize functions by domain, range, and intent — never by implementation; any implementation with the same attributes is interchangeable.
5. Infix operators are shorthand for call expressions; prefer operators and parentheses for simple math; know `/` vs `//`.
6. `def` and `=` both bind names and destroy previous bindings; intrinsic names play no role in evaluation.

## Connects To
- **Section 1.2**: extends the call-expression evaluation procedure from built-ins to user-defined functions.
- **Section 1.4**: domain/range/intent become documentation and design guidelines.
- **Section 1.5.3**: assignment inside bodies extends the local-frame story.
- **Section 1.6**: nested definitions add parent environments to frames; this model is the prerequisite.
- **Chapter 3**: this model of evaluation is the blueprint for implementing a working interpreter.
