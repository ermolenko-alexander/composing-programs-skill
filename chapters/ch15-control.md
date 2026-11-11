# Section 1.5: Control

## Core Idea
Control statements have no value — executing one determines what the interpreter should do next, letting programs make comparisons, choose branches, and repeat statements.

## Frameworks Introduced
- **Compound statement anatomy**: a one-line *header* ending in a colon plus an indented *suite* of statements is a *clause*; a compound statement is one or more clauses. "The header controls its suite" — specialized evaluation rules per header kind dictate when/if the suite executes.
- **Execution rule for a sequence of statements**: execute the first statement; if it does not redirect control, execute the rest. This exposes sequences as recursively defined (first + rest), and means later statements may never be reached.
- **Execution procedure for conditional statements**: a required `if` clause, optional `elif` clauses, optional `else`. Consider each clause in order: evaluate the header expression; if a true value, execute the suite and skip all subsequent clauses. `else` runs only if all earlier expressions were false.
- **Execution procedure for `while`**: (1) evaluate the header expression; (2) if true, execute the entire suite, then return to step 1. The suite should always change some binding each pass, or you get an infinite loop (Control-C stops it).
- **Short-circuiting evaluation procedures**: `<left> and <right>` — evaluate left; if false value v, result is v; otherwise result is right's value. `<left> or <right>` — evaluate left; if true value v, result is v; otherwise right's value. `not <exp>` — True if the result is a false value, False otherwise.
- **Testing discipline**: write (and run) tests immediately after implementing new functions — even before implementing, to fix example inputs/outputs in mind. A test applying a single function is a *unit test*; exhaustive unit testing is a hallmark of good program design. Mechanisms: `assert` statements, test functions in `_test.py` files, and *doctests* (sample interactive sessions in docstrings, run via `doctest.testmod()`, `run_docstring_examples(fn, globals(), True)`, or `python3 -m doctest <file>`).

## Key Concepts
- **Control statement**: a statement that controls the flow of execution based on logical comparisons; it has no value.
- **Executed vs evaluated**: statements are executed (they change interpreter state); an expression used as a statement is evaluated and its value discarded.
- **Local assignment**: binds a name in the *first* frame of the current environment — so assignments inside a function body cannot affect the global frame; this is what makes modular programs possible, with pure functions interacting only via values taken and returned.
- **Return statement redirects control**: function application terminates at the first `return` executed.
- **Boolean context**: a header expression whose truth value matters to control flow; false values include 0, `None`, `False` — all other numbers are true values.
- **Boolean values**: `True` and `False`, returned by comparisons `>, <, >=, <=, ==, !=` (`==` is comparison, `=` is assignment).
- **Assertion**: `assert <expr>, 'message'` — no effect if true, halts with an error if false.
- **Doctest**: an interactive session embedded in a docstring that doubles as test and documentation.

## Mental Models
- Think of each compound statement as header-controls-suite: to learn a new statement, learn its header's execution rule.
- Use simultaneous assignment (`pred, curr = curr, pred + curr`) when a state update depends on old values: everything right of `=` evaluates before any rebinding — essential for correctness.
- Think of naming conventions as signals: boolean-returning functions begin with `is` not followed by an underscore (`isfinite`, `isinstance`).
- Use local assignment to name intermediate quantities and clarify complex expressions.

## Anti-patterns
- **Expression statement without `return`**: `def square(x): mul(x, x)` is valid Python but discards the result — if you want the result, store or return it.
- **Loop body that changes no binding**: guarantees an infinite loop.
- **Confusing `=` and `==`**: assignment vs equality comparison.
- **Testing only typical inputs**: a test function should cover several arguments including extreme values (e.g. `fib(50)`).
- **Mixing tabs and spaces / uneven indentation**: any variation in suite indentation causes an error.

## Code Examples
```python
def fib(n):
    """Compute the nth Fibonacci number, for n >= 2."""
    pred, curr = 0, 1   # Fibonacci numbers 1 and 2
    k = 2               # Which Fib number is curr?
    while k < n:
        pred, curr = curr, pred + curr
        k = k + 1
    return curr
```
- **What it demonstrates**: `while` execution, simultaneous assignment for state updates, and loop-variable bookkeeping.

## Worked Example
`fib(8)` tracks three bindings: `pred` (predecessor), `curr` (the kth Fibonacci number), and counter `k`. Starting at `pred, curr = 0, 1`, `k = 2`, each pass executes `pred, curr = curr, pred + curr` — both right-side expressions evaluate against the old bindings before either name is rebound — and increments `k`, until `k < n` fails; it returns `curr = 13`. Tested as `assert fib(8) == 13, 'The 8th Fibonacci number should be 13'`, and a `fib_test()` function additionally checks `fib(2)`, `fib(3)`, and `fib(50) == 7778742049`.

## Key Takeaways
1. Learn each compound statement as a header rule: conditionals pick the first true clause; `while` re-checks after each full suite execution.
2. `and`/`or` short-circuit and return one of their operand values, not necessarily a boolean.
3. Local assignment cannot touch the global frame — this isolation is what makes functions modular.
4. Simultaneous assignment evaluates all right-side values before rebinding; use it for interdependent state updates.
5. Write doctests/asserts immediately after (or before) implementing; run with `python3 -m doctest file.py`.
6. Every loop must change a binding its header depends on.

## Connects To
- **Section 1.2**: refines the statement/expression distinction (executed vs evaluated) and the multiple-assignment rule.
- **Section 1.6**: `while`-based summation templates (`sum_naturals`) become the patterns that higher-order functions abstract.
- **Section 1.7**: iteration here contrasts with recursion as the other repetition mechanism.
