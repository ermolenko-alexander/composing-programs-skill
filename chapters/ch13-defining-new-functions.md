# Section 1.3: Defining New Functions

## Core Idea
Function definitions are a much more powerful abstraction technique than assignment: a name is bound to a compound operation that can be referred to as a unit, and a precise environment model of frames explains exactly how such functions evaluate.

## Frameworks Introduced
- **How to define a function**: a `def` statement with a `<name>` and `<formal parameters>`, then an indented body ending in a `return <return expression>`. The return expression is *not* evaluated right away — it is stored and evaluated only when the function is applied.
- **Environment diagrams**: the environment is a sequence of *frames* (boxes) containing *bindings* (name→value). There is a single *global frame*; assignment and import add entries to the first frame of the current environment.
  - When to use: whenever the meaning of a program is non-obvious (name shadowing, repeated calls, local vs global names).
  - How: step through the code binding by binding; draw each function as `func name(params)`, each call as a new frame.
- **Procedure for applying a user-defined function**: (1) bind the arguments to the names of the formal parameters in a *new local frame*; (2) execute the body in the environment that starts with this frame. The environment is then [local frame, global frame]; each instance of a function application has its own independent local frame.
- **Name Evaluation rule**: "A name evaluates to the value bound to that name in the earliest frame of the current environment in which that name is found."
- **Functions as abstractions / aspects of a functional abstraction**: master any function via three attributes — *domain* (set of arguments it can take), *range* (set of values it can return), *intent* (the input→output relationship plus side effects). The implementation details are suppressed; any function with the same attributes is interchangeable ("black box").
- **Operators as shorthand**: infix expressions are shorthand for call expressions (`2 + 3` ≈ `add(2, 3)`); precedence and parentheses determine the implicit nesting; `/` is `truediv` (always float), `//` is `floordiv` (rounds down: `-5 // 4 == -2`).

## Key Concepts
- **Formal parameter**: the name in a definition for the thing to be operated on; its choice must not affect behavior.
- **Frame**: a box of bindings; local frames keep same-named parameters in different calls separate.
- **Global frame**: the single shared frame holding imports, defs, and top-level assignments.
- **Local frame**: created fresh at every function application, even for repeated calls of the same function.
- **Intrinsic name vs bound name**: a function's own name (shown in `func …`) vs the frame name(s) referring to it; many names can refer to one function, and only the bound name matters in evaluation.
- **Function signature**: the description of a function's formal parameters (built-ins render as `name(...)`).
- **Scope**: a local name is limited to the body of the function that defines it — a consequence of how environments work, not an extra rule.
- **Model of evaluation**: the conceptual framework of environments, names, and functions that precisely describes how the interpreter evaluates call expressions.

## Mental Models
- Think of each function call as opening a fresh private workspace (local frame); `x` in `square` and `x` in `sum_squares` can never collide.
- Use the "earliest frame wins" rule to resolve any shadowing question mechanically.
- Think of a function definition as a stored promise: the body runs at call time, never at definition time.
- Think of a used function as a black box defined only by domain, range, and intent — "a programmer should not need to know how the function is implemented in order to use it."

## Anti-patterns
- **Relying on parameter names from outside**: the meaning of a function must be independent of the parameter names its author chose.
- **Confusing intrinsic and bound names**: after `f = max; max = 3`, calling `max(1, 2)` gives `TypeError: 'int' object is not callable` while `f` still works.
- **Expecting the body to run at `def` time**: the entire `def` statement is processed in a single step; the body executes only on call.
- **Poor names**: function names should evoke the operation (`print`, `add`) or resulting quantity (`max`, `sum`); parameters should evoke their role; avoid `l`, `O`, `I` (PEP-8 style guide).

## Code Examples
```python
>>> from operator import add, mul
>>> def square(x):
        return mul(x, x)
>>> def sum_squares(x, y):
        return add(square(x), square(y))
>>> sum_squares(3, 4)
25
```
- **What it demonstrates**: user-defined functions used exactly like built-ins and composed as building blocks; one cannot tell from `sum_squares` whether `square` is built-in, imported, or user-defined.

## Worked Example
Evaluating `result = sum_squares(5, 12)`: Python evaluates the name `sum_squares` (bound in the global frame) and the numerals 5, 12; applies it, creating a local frame binding x=5, y=12. The body `add(square(x), square(y))` evaluates all three subexpressions in the environment starting at the `sum_squares` frame: `add` is found in the global frame; operand 0 applies `square` to 5 in *yet another* local frame (x=5), where `mul(x, x)` → 25; operand 1 creates a third local frame (x=12) → 144; `add(25, 144)` → 169. Three local frames each bind `x` to different values — the frame machinery exists exactly so names resolve to the correct values at the correct times.

## Key Takeaways
1. Apply user-defined functions by the two-step rule: bind arguments in a new local frame, then execute the body in [local, global].
2. Resolve any name by the Name Evaluation rule: earliest frame in the current environment wins.
3. Every call gets an independent local frame; parameter names are local and interchangeable.
4. Characterize functions you use by domain, range, and intent — never by implementation.
5. Infix operators are readable shorthand for call expressions; idiomatic Python prefers operators for simple arithmetic.
6. `def` and `=` both bind names and destroy previous bindings.

## Connects To
- **Section 1.2**: extends the call-expression evaluation procedure to user-defined operators.
- **Section 1.6**: nested definitions extend frames with parent environments; the environment model built here is the prerequisite.
- **Chapter 3**: this model of evaluation is the blueprint for implementing an interpreter.
