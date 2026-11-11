# Section 1.2: Elements of Programming

## Core Idea
A programming language is more than a means for instructing a computer — it is a framework within which we organize ideas about computational processes, and programs serve to communicate those ideas among a community. Thus "programs must be written for people to read, and only incidentally for machines to execute." Every powerful language provides primitives, means of combination, and means of abstraction — for both functions and data.

## Frameworks Introduced

### The Three Mechanisms of Every Powerful Language
1. **Primitive expressions and statements** — the simplest building blocks the language provides.
2. **Means of combination** — by which compound elements are built from simpler ones.
3. **Means of abstraction** — by which compound elements can be named and manipulated as units.

In programming we deal with two kinds of elements: **functions** and **data** ("soon we will discover they are really not so distinct"). Informally, data is stuff we want to manipulate; functions describe the rules for manipulating the data. Any powerful language must describe primitive data and primitive functions, plus methods for combining and abstracting *both*.
- When to use: evaluating any language or DSL, or deciding what a new abstraction layer must provide.
- How: check that primitives, combination, and abstraction exist for both functions and data.

### Pure vs Non-Pure Functions
- **Pure functions**: take input (arguments) and return output; applying them has *no effects beyond returning a value*, and a pure function must always return the same value when called twice with the same arguments. Picture: a small machine that takes input and produces output (`abs(-2)` → `2`).
- **Non-pure functions**: in addition to returning a value, applying one can generate **side effects** — changes to the state of the interpreter or computer. The common example is `print`, whose side effect is generating output; its return value is always `None`.
- Three benefits of purity, per the author: (1) pure functions compose more reliably into compound call expressions — `print` returns no useful result as an operand, whereas `max`, `pow`, `sqrt` nest effectively; (2) pure functions are simpler to test — a list of arguments always leads to the same return value; (3) pure functions are essential for *concurrent* programs (Chapter 4), where multiple call expressions may be evaluated simultaneously. The rest of Chapter 1 concentrates on pure functions; `print` is used only to see intermediate results.

## Evaluation Rules (the author's formal procedures, verbatim-faithful)

**Procedure for evaluating a call expression.** To evaluate a call expression, Python will do the following:
1. Evaluate the operator and operand subexpressions, then
2. Apply the function that is the value of the operator subexpression to the arguments that are the values of the operand subexpressions.

The first step dictates that to evaluate a call expression we must first evaluate *other* expressions — so the evaluation procedure is **recursive** in nature: it includes, as one of its steps, the need to invoke the rule itself.

**Primitive cases** (where the recursion bottoms out):
- A numeral evaluates to the number it names.
- A name evaluates to the value associated with that name in the current environment.

**Execution of an assignment statement.** When executing an assignment statement, Python evaluates the expression to the right of `=` *before* changing the binding to the name on the left (so the right side may mention the very name being bound: `x = x + 1`). With **multiple assignment**, *all* expressions to the right of `=` are evaluated before *any* names on the left are bound to those values — which is why `y, x = x, y` swaps in a single statement.

**Statements vs expressions.** Executing `x = 3` does not return a value nor evaluate a function on arguments; its purpose is to bind a name to a value. In general, statements are not evaluated but *executed*; they do not produce a value but instead make some change. Each type of expression or statement has its own evaluation or execution procedure.

**Pedantic note** (the author's): when we say "a numeral evaluates to a number," we actually mean the Python *interpreter* evaluates a numeral to a number — it is the interpreter which endows meaning to the language. Since the interpreter is a fixed program that behaves consistently, we may speak of expressions themselves evaluating to values.

## Key Concepts
- **Primitive expression**: e.g., a number — strictly, the numerals you type represent the number in base 10.
- **Infix notation**: operator (`+ - * /`) between operands; one of many compound-expression forms.
- **Call expression**: the most important compound expression — applies a function to arguments; the *operator* expression precedes parentheses enclosing a comma-delimited list of *operand* expressions.
- **Operator / operands / arguments / return value**: the operator specifies a *function*; when evaluated, the function is *called* with *arguments* and *returns* a *value*. Argument order matters: `pow(100, 2)` = 10000, `pow(2, 100)` = 1267650600228229401496703205376.
- **Three advantages of function notation over infix**: (1) functions may take an arbitrary number of arguments with no ambiguity — the function name always precedes its arguments; (2) it extends straightforwardly to *nested* expressions where nesting structure is entirely explicit in the parentheses; (3) it unifies mathematics' zoo of notations (superscript exponents, bars, radicals — "very hard to type") — any operator can be expressed as a function with a name.
- **Module / import**: Python organizes functions into modules comprising the Python Library; an import statement designates a module name and lists the named attributes to import. `operator` module functions (`add`, `sub`, `mul`) are identical in behavior to the infix symbols; convention favors symbols for simple arithmetic.
- **Binding / name**: if a value has been given a name, the name *binds* to the value. Bindings are established by assignment and by import statements. Names are called *variables* because they can be bound to different values over a program's execution.
- **Environment**: the interpreter's memory that keeps track of names, values, and bindings. Environments provide the context in which evaluation takes place — it is *meaningless* to speak of the value of `add(x, 1)` without an environment giving meaning to `x` (or even `add`).
- **Assignment operator `=`**: the simplest means of abstraction — simple names refer to results of compound operations; complex programs are built step by step from computational objects of increasing complexity.
- **Names bound to functions**: `f = max` makes `f(2, 3, 4)` → 4; Python prints `<built-in function max>` as a function's description. Rebinding is total: after `max = 5`, `max` is no longer bound to a function and calling it errors.
- **Expression tree**: trees grow top-down; nodes are expressions paired with their values; leaves are functions or numbers; interior nodes hold a call expression and its result; operand values *percolate upward* from terminal nodes, combining at higher and higher levels.
- **`None`**: the special Python value representing nothing; `print` always returns it; the interactive interpreter does not auto-display it.
- **Side effect**: a change to interpreter/computer state beyond returning a value.

## Mental Models
- **Function notation as the universal form**: any infix operator is a function with a name; call notation removes all ambiguity.
- **Expression tree for surprises**: whenever a nested expression's output surprises you (e.g., `print(print(1), print(2))`), the author's advice is literally "draw an expression tree to clarify."
- **Rebinding is total**: binding a name to a new value severs it from any previous value — even built-ins.
- **Assignment as snapshot**: `area` computed from `radius` does not update when `radius` changes; only another assignment updates it.
- **Experiment over documentation** (early on): "experimenting with a function tells you more about its behavior than reading the documentation" until you know the language well.

## Anti-patterns
- **Assigning the result of `print`**: `two = print(2)` prints `2` and binds `two` to `None`. The author: "Be careful with print! The fact that it returns None means that it *should not* be the expression in an assignment statement."
- **Composing non-pure functions**: they don't return useful results as operands; only pure functions compose reliably into compound call expressions.
- **Shadowing built-in names**: `max = 5` makes `max(2, 3, 4)` an error.
- **Deep multi-level nesting**: no limit in principle to depth the interpreter handles, but "humans quickly get confused by multi-level nesting" — an important role for the programmer is structuring expressions so they remain interpretable by yourself, partners, and future readers.
- **Expecting dependent names to track**: `radius = 11` leaves `area` at its old value.

## Code Examples
```python
>>> 42
42
>>> -1 - -1
0
>>> 1/2 + 1/4 + 1/8 + 1/16 + 1/32 + 1/64 + 1/128
0.9921875

>>> max(7.5, 9.5)
9.5
>>> max(1, -2, 3, -4)
3
>>> max(min(1, -2), min(pow(3, 5), -4))
-2

>>> from math import sqrt
>>> sqrt(256)
16.0
>>> from operator import add, sub, mul
>>> add(14, 28)
42
>>> sub(100, mul(7, add(8, 4)))
16
```
Names and the environment:
```python
>>> radius = 10
>>> 2 * radius
20
>>> from math import pi
>>> pi * 71 / 223
1.0002380197528042
>>> f = max
>>> f(2, 3, 4)
4
>>> f = 2            # rebinding
>>> max = 5          # even built-ins can be rebound (then max(2,3,4) errors)
>>> x = 2
>>> x = x + 1        # right side evaluated before binding
>>> x
3
>>> area, circumference = pi * radius * radius, 2 * pi * radius
>>> x, y = 3, 4.5
>>> y, x = x, y      # swap in one statement
```
Non-pure `print`:
```python
>>> print(1, 2, 3)
1 2 3
>>> print(print(1), print(2))
1
2
None None
>>> two = print(2)
2
>>> print(two)
None
```

## Worked Examples

### 1. Evaluating `sub(pow(2, add(1, 10)), pow(2, 5))` → 2016
The evaluation procedure is applied four times. Draw the expression tree: the root is the full `sub(...)` call; its branches are the operator `sub` and two `pow` calls; `pow(2, add(1, 10))` branches further into `add(1, 10)`.
- Leaves: names (`sub`, `pow`, `add` — each evaluates to its function in the current environment) and numerals (2, 1, 10, 5 — each to the number it names).
- `add(1, 10)` → 11; then `pow(2, 11)` → 2048; `pow(2, 5)` → 32; finally `sub(2048, 32)` → 2016.
- Values of operands percolate upward from terminal nodes, combining at higher levels. Repeated application of step 1 is exactly the recursion.

### 2. Why `print(print(1), print(2))` prints three lines
Step 1 evaluates operator (`print`, the function) and operands. Evaluating operand `print(1)` *as a side effect* outputs `1` and evaluates to `None`; likewise `print(2)` outputs `2` and yields `None`. Step 2 applies the outer `print` to arguments `None, None`, which outputs `None None` as its own side effect and itself returns `None` (not displayed). The tree makes the order and the `None` values obvious — this is the canonical demonstration that non-pure functions don't compose.

### 3. The swap, via the multiple-assignment rule
`x, y = 3, 4.5` then `y, x = x, y`: both right-side expressions (`x` → 3, `y` → 4.5) are evaluated *before* any left-side binding changes, so afterwards `x` is 4.5 and `y` is 3. Without the all-rights-before-any-lefts rule, a temporary name would be needed.

## Common Errors (explicitly warned in the text)
- `two = print(2)` — binding a name to `None` because `print` was used for its (nonexistent) value.
- `max = 5` followed by `max(2, 3, 4)` — calling a name no longer bound to a function causes an error.
- Expecting `area` to change when `radius` does — updating requires another assignment statement.
- Evaluating `add(x, 1)` with no environment binding for `x` — meaningless without context; names only have values in an environment.
- Confusing evaluation with execution — assignment neither returns a value nor applies a function.

## Key Takeaways
1. Judge every language by its primitives, means of combination, and means of abstraction — for functions *and* data.
2. Memorize the two-step call-expression procedure; it is recursive and bottoms out at two stipulations: numerals name numbers, names look up values in the current environment.
3. Names get meaning only from an environment — evaluation always happens in a context.
4. Prefer pure functions: reliable composition, simple testing, concurrency-readiness; use `print` only to observe intermediates.
5. In (multiple) assignment, all right-side expressions evaluate before any binding changes.
6. Statements are executed for effect; expressions are evaluated for value — each form has its own procedure.

## Connects To
- **Section 1.3**: `def` supplies the much more powerful means of abstraction (binding names to compound operations), forcing the full environment model.
- **Section 1.1**: formalizes the expressions/statements split previewed in the Shakespeare example.
- **Section 1.6**: pure-function composition becomes the basis for higher-order functions.
- **Chapter 2**: investigates non-pure functions and data; Chapter 4: purity and concurrency.
