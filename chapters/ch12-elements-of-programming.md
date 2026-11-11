# Section 1.2: Elements of Programming

## Core Idea
Programs must be written for people to read, and only incidentally for machines to execute; every powerful language provides primitives, means of combination, and means of abstraction for both functions and data.

## Frameworks Introduced
- **The three mechanisms of every powerful language**: (1) *primitive expressions and statements* — the simplest building blocks; (2) *means of combination* — compound elements built from simpler ones; (3) *means of abstraction* — compound elements named and manipulated as units.
  - When to use: evaluating any language or DSL, or deciding what a new abstraction layer must provide.
  - How: check that primitives, combination, and abstraction exist for both functions and data.
- **Evaluation procedure for call expressions**: To evaluate a call expression, Python will (1) evaluate the operator and operand subexpressions, then (2) apply the function that is the value of the operator subexpression to the arguments that are the values of the operand subexpressions.
  - When to use: predicting the result of any nested expression; the procedure is *recursive* — it invokes itself on subexpressions.
  - How: base cases: a numeral evaluates to the number it names; a name evaluates to the value associated with it in the current environment.
- **Pure functions vs non-pure functions**: pure functions return a value and have no effects beyond that, and always return the same value for the same arguments; non-pure functions also generate *side effects* (e.g. `print` outputs text and returns `None`).
  - When to use: deciding how composable, testable, and concurrency-safe a function is.
  - How: prefer pure functions; use non-pure ones (like `print`) only to observe intermediate results.
- **Expression tree**: draw a nested expression as a tree whose leaves are numbers/functions and whose interior nodes pair call expressions with their values; values "percolate upward" from the leaves.

## Key Concepts
- **Call expression**: the most important compound expression — an operator expression followed by parenthesized, comma-delimited operand expressions.
- **Operator / operands**: the function-specifying expression and its argument expressions.
- **Binding**: association of a name with a value, created by assignment or import statements.
- **Environment**: the interpreter's memory tracking names, values, and bindings; it provides the context without which an expression like `add(x, 1)` is meaningless.
- **Assignment operator (`=`)**: the simplest means of abstraction — simple names refer to results of compound operations; the right side is fully evaluated before the name is bound.
- **Multiple assignment**: all right-hand expressions evaluate before any left-hand name is bound, so `y, x = x, y` swaps in one statement.
- **Side effect**: a change to interpreter or computer state beyond returning a value.
- **None**: the special value representing nothing, returned by `print` and not auto-displayed interactively.
- **Evaluated vs executed**: expressions are evaluated to values; statements are executed and produce no value, they make a change.

## Mental Models
- Think of function notation as the universal form: any infix operator is a function with a name (`add(2, 3)`), and call notation removes all ambiguity because the function name always precedes its arguments.
- Use an expression tree whenever a nested expression's output surprises you (e.g. `print(print(1), print(2))`).
- Think of rebinding as total: when a name is bound to a new value, it is no longer bound to any previous value — even built-ins (`max = 5` destroys access to the function).
- Think of assignment as snapshotting: `area` computed from `radius` does not update when `radius` changes; only re-assignment updates it.

## Anti-patterns
- **Assigning the result of `print`**: `two = print(2)` binds `two` to `None` — never use a `None`-returning function as the expression in an assignment.
- **Composing non-pure functions in nested expressions**: they don't return useful results as operands; only pure functions compose reliably.
- **Shadowing built-in names**: rebinding `max` to a number makes `max(2, 3, 4)` raise an error.
- **Deep multi-level nesting**: the interpreter handles unlimited depth, but humans get confused — structure expressions for readers.

## Code Examples
```python
>>> from operator import add, sub, mul
>>> sub(100, mul(7, add(8, 4)))
16
>>> print(print(1), print(2))
1
2
None None
```
- **What it demonstrates**: the recursive evaluation procedure on nested calls, and the non-pure character of `print` (side-effect output, `None` return).

## Worked Example
Evaluating `sub(pow(2, add(1, 10)), pow(2, 5))` applies the evaluation procedure four times. Draw the expression tree: leaves are the functions (`sub`, `pow`, `add`) and numerals (2, 1, 10, 5). `add(1, 10)` evaluates first to 11; then `pow(2, 11)` to 2048; `pow(2, 5)` to 32; finally `sub(2048, 32)` yields 2016. Values percolate upward from terminal nodes, combining at higher levels — the procedure is recursive because step 1 (evaluate subexpressions) re-invokes the whole rule.

## Key Takeaways
1. Judge every language by its primitives, means of combination, and means of abstraction.
2. Memorize the two-step evaluation procedure for call expressions; it is recursive and bottoms out at numerals and names.
3. Names get meaning only from an environment — evaluation always happens in a context.
4. Prefer pure functions: they compose into nested expressions, are simpler to test, and are essential for concurrency.
5. In multiple assignment, all right-side expressions evaluate before any binding changes.
6. Statements are executed for effect; expressions are evaluated for value — keep the distinction sharp.

## Connects To
- **Section 1.3**: `def` adds the missing powerful means of abstraction (function definition) and forces a precise environment model with frames.
- **Section 1.1**: formalizes the expressions/statements split previewed in the first example.
- **Section 1.6**: pure-function composition becomes the basis for higher-order functions.
