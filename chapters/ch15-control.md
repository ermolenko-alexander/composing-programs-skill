# Section 1.5: Control

## Core Idea
The functions we can define so far are limited because we cannot make comparisons and act on their results. **Control statements** give this ability: they control the flow of a program's execution based on the results of logical comparisons. They differ fundamentally from expressions: "They have no value. Instead of computing something, executing a control statement determines what the interpreter should do next."

## Frameworks Introduced

### Statements vs Expressions (refined)
We have already seen three statements: assignment, `def`, and `return`. They are not expressions, though they contain expressions as components. Statements are *executed*, not evaluated: each describes a change to interpreter state, and executing it applies that change. Expressions can also be executed as statements, in which case they are evaluated but **their value is discarded** — executing a pure function thus has no effect; executing a non-pure one can cause effects. "Statements govern the relationship among different expressions in a program and what happens to their results."

### Compound Statement Anatomy
A **simple statement** is one line not ending in a colon. A **compound statement** spans multiple lines, starts with a one-line **header** ending in a colon identifying the statement's type, followed by an indented **suite** of statements; header + suite = a **clause**; a compound statement is one or more clauses:
```
<header>:
    <statement>
    <statement>
    ...
<separating header>:
    <statement>
    ...
```
Expressions, `return`, and assignment are simple statements; `def` is a compound statement whose suite is the function body. Specialized evaluation rules per header kind dictate when and if the suite executes — **"the header controls its suite"** (e.g., a `def` header stores its return expression for later rather than evaluating it).
**Practical guidance**: all suite lines must be indented the same amount and in the same way (use spaces, not tabs); any variation in indentation causes an error.

### Local Assignment and Modularity
Function bodies may be whole sequences of clauses, executed in a local environment starting with the call's local frame. The effect of an assignment statement is to bind a name in the **first** frame of the current environment — so **assignments within a function body cannot affect the global frame**. "The fact that functions can only manipulate their local environment is critical to creating *modular* programs, in which pure functions interact only via the values they take and return." Local assignment also clarifies complex expressions by naming intermediate quantities.

### Testing Discipline
Testing verifies that a function's behavior matches expectations. A *test* is typically another function containing sample calls with specific argument values, verified against expected results; tests also serve as documentation of how to call a function. Mechanisms:
- **Assertions**: `assert <expr in boolean context>, 'message'` — no effect if true; if false, an error halts execution and the message is displayed.
- **Test functions**: should test several arguments, *including extreme values*; in files, tests live in the same file or a neighboring `_test.py` file.
- **Doctests**: a sample interactive session in the docstring (after the one-line description and a blank line). Verify via `from doctest import testmod; testmod()`, or for one function `run_docstring_examples(fn, globals(), True)` (arg 2 must be `globals()` — the global environment the interpreter needs; arg 3 `True` means verbose), or from the shell: `python3 -m doctest <python_source_file>`.
- **The key to effective testing**: "write (and run) tests immediately after implementing new functions. It is even good practice to write some tests before you implement, in order to have some example inputs and outputs in your mind." A test applying a single function is a **unit test**; "exhaustive unit testing is a hallmark of good program design."

## Evaluation Rules (the author's formal procedures, verbatim-faithful)

**Executing a sequence of statements.** To execute a sequence of statements, execute the first statement. If that statement does not redirect control, then proceed to execute the rest of the sequence of statements, if any remain. — This exposes a sequence as recursively defined (first + rest; the rest is itself a sequence), a view that reappears in later chapters. The important consequence: statements execute in order, but later statements may never be reached, because of redirected control.

**Return redirects control.** Function application terminates whenever the first `return` statement is executed, and the value of the return expression is the returned value of the function being applied.

**Executing a conditional statement.** Form: a required `if` clause, an optional sequence of `elif` clauses, and finally an optional `else` clause:
```
if <expression>:
    <suite>
elif <expression>:
    <suite>
else:
    <suite>
```
Each clause is considered in order. The computational process of executing a conditional clause:
1. Evaluate the header's expression.
2. If it is a true value, execute the suite. Then skip over all subsequent clauses in the conditional statement.
If the `else` clause is reached (only when all `if`/`elif` expressions evaluated to false values), its suite is executed.

**Executing a `while` clause.** Form: `while <expression>:` followed by a suite.
1. Evaluate the header's expression.
2. If it is a true value, execute the suite, then return to step 1.
In step 2, the *entire* suite is executed before the header expression is evaluated again. To prevent indefinite execution, the suite should always change some binding in each pass. A `while` statement that does not terminate is an **infinite loop**; press `<Control>-C` to force Python to stop looping.

**Short-circuiting logical operators.** The truth value of a logical expression can sometimes be determined without evaluating all subexpressions — *short-circuiting*:
- To evaluate `<left> and <right>`: (1) evaluate `<left>`; (2) if the result is a false value `v`, the expression evaluates to `v`; (3) otherwise it evaluates to the value of `<right>`.
- To evaluate `<left> or <right>`: (1) evaluate `<left>`; (2) if the result is a true value `v`, the expression evaluates to `v`; (3) otherwise it evaluates to the value of `<right>`.
- To evaluate `not <exp>`: evaluate `<exp>`; the value is `True` if the result is a false value, and `False` otherwise.

**Multiple assignment order of events**: all expressions to the right of `=` are evaluated before any rebinding takes place — "this order of events ... is essential for correctness" of `fib`.

## Key Concepts
- **Control statement**: flow control based on logical comparisons; valueless.
- **Boolean context**: the header expressions of conditional blocks — their truth values matter to control flow but are not assigned or returned. False values include `0`, `None`, and `False`; **all other numbers are true values** (Chapter 2: every built-in kind of data has both true and false values).
- **Boolean values**: `True` and `False`, returned by the comparisons `>, <, >=, <=, ==, !=` (corresponding to `operator` functions like `ge`, `eq`). Python distinguishes assignment `=` from equality comparison `==`.
- **Boolean operators**: `and`, `or`, `not` — they return *operand values*, not necessarily booleans.
- **Naming convention**: functions that perform comparisons and return booleans typically begin with `is`, not followed by an underscore (`isfinite`, `isdigit`, `isinstance`).
- **Clause / header / suite**: the compound-statement vocabulary.
- **Infinite loop**: a non-terminating `while`.
- **Assertion / unit test / doctest**: the three testing mechanisms.
- **Expression statement**: an expression executed alone; value discarded — only useful for non-pure calls like `print`.

## Mental Models
- To learn any new compound statement, learn its header's execution rule — the header controls its suite.
- Use simultaneous assignment (`pred, curr = curr, pred + curr`) whenever a state update depends on old values: everything right of `=` evaluates first.
- `and`/`or` are value-selecting, short-circuiting operators, not boolean-producing ones.
- Local assignment is a scalpel for readability: name intermediate quantities (`difference = abs(x-y)`).
- Treat a loop's bindings as the loop's state machine: the header expression must depend on something the suite changes.

## Anti-patterns
- **Expression statement where a return was intended** (explicit warning in the text):
  ```python
  >>> def square(x):
          mul(x, x)  # Watch out! This call doesn't return a value.
  ```
  Valid Python, but the result is discarded. "If you want to do something with the result of an expression, you need to say so" — store it with assignment or return it. A body that is only an expression does make sense for non-pure calls: `def print_square(x): print(square(x))`.
- **Loop suite that changes no binding**: guarantees an infinite loop.
- **Confusing `=` and `==`**: assignment vs equality comparison.
- **Testing only typical inputs**: test several arguments including extremes (`fib(50)`).
- **Mixing tabs and spaces / uneven indentation**: any variation causes an error.
- **Sequential instead of simultaneous update**: rebinding `pred` before computing `pred + curr` would corrupt `fib`.

## Code Examples
Conditional (the motivating example — implementing `abs` ourselves):
```python
def absolute_value(x):
    """Compute abs(x)."""
    if x > 0:
        return x
    elif x == 0:
        return 0
    else:
        return -x

result = absolute_value(-2)
```
Local assignment:
```python
def percent_difference(x, y):
    difference = abs(x-y)
    return 100 * difference / x
result = percent_difference(40, 50)   # 25.0
# equivalent single-expression form (more complex return expression):
def percent_difference(x, y):
    return 100 * abs(x-y) / x
```
Iteration:
```python
def fib(n):
    """Compute the nth Fibonacci number, for n >= 2."""
    pred, curr = 0, 1   # Fibonacci numbers 1 and 2
    k = 2               # Which Fib number is curr?
    while k < n:
        pred, curr = curr, pred + curr
        k = k + 1
    return curr

result = fib(8)   # 13
```
Testing:
```python
>>> assert fib(8) == 13, 'The 8th Fibonacci number should be 13'
>>> def fib_test():
        assert fib(2) == 1, 'The 2nd Fibonacci number should be 1'
        assert fib(3) == 1, 'The 3rd Fibonacci number should be 1'
        assert fib(50) == 7778742049, 'Error at the 50th Fibonacci number'

>>> def sum_naturals(n):
        """Return the sum of the first n natural numbers.

        >>> sum_naturals(10)
        55
        >>> sum_naturals(100)
        5050
        """
        total, k = 0, 1
        while k <= n:
            total, k = total + k, k + 1
        return total

>>> from doctest import testmod
>>> testmod()
TestResults(failed=0, attempted=2)
>>> from doctest import run_docstring_examples
>>> run_docstring_examples(sum_naturals, globals(), True)
```
Booleans:
```python
>>> 4 < 2        # False
>>> 5 >= 5       # True   (operator.ge)
>>> 0 == -0      # True   (operator.eq)
>>> True and False   # False
>>> True or False    # True
>>> not False        # True
```

## Worked Examples

### 1. `absolute_value(-2)` through the conditional procedure
Clauses are considered in order. Header `x > 0`: evaluates `-2 > 0` → `False` (a false value) — skip the suite. Header `x == 0`: `False` — skip. The `else` clause is reached because all earlier expressions were false; its suite executes `return -x` → 2, which redirects control, terminating the application. Note the boolean-context point: the header values are consulted for truth and then discarded.

### 2. `fib(8)` → 13, with the simultaneous-assignment rule doing the real work
State: `pred, curr = 0, 1`, `k = 2` (curr is the 2nd Fibonacci number). Each `while` pass: evaluate `k < n`; if true, execute the *entire* suite — `pred, curr = curr, pred + curr` evaluates *both* right-side expressions against the old bindings before either name is rebound (rebinding pred to curr's value and simultaneously curr to pred+curr), then `k = k + 1` — then return to re-evaluate the header. Passes take (pred, curr) through (1,1) k=3, (1,2) k=4, (2,3) k=5, (3,5) k=6, (5,8) k=7, (8,13) k=8; now `k < 8` is false and `return curr` yields 13. Verified by `assert fib(8) == 13, ...`, and `fib_test()` additionally checks the edge `fib(2)`, `fib(3)`, and extreme `fib(50) == 7778742049`.

### 3. `sum_naturals` with doctests, end to end
Docstring begins with a one-line description, a blank line, then a sample session. `testmod()` runs all doctests in the module → `TestResults(failed=0, attempted=2)`. `run_docstring_examples(sum_naturals, globals(), True)` tests just this function verbosely, printing each Trying/Expecting/ok block; a mismatched return value is reported as a test failure. From files: `python3 -m doctest file.py`.

## Common Errors (what the author explicitly warns about)
- The `# Watch out!` example: a function body consisting of a bare expression discards its result and returns `None` implicitly.
- Infinite loops from suites that change no binding the header depends on (`<Control>-C` to stop).
- Indentation variation within a suite (and tabs instead of spaces) — an error.
- `=` vs `==` confusion — a convention shared across many languages.
- Forgetting that in `fib`, evaluating right-side expressions before any rebinding is *essential for correctness* — a sequential rewrite breaks it.

## Key Takeaways
1. Learn each compound statement as a header rule: conditionals pick the first true clause (else only if all were false); `while` re-checks its header after each full suite execution.
2. `and`/`or` short-circuit and return one of their operand values; `not` always returns a boolean.
3. False values include 0, `None`, `False`; all other numbers are true — headers sit in boolean contexts.
4. Local assignment binds in the first frame of the current environment and cannot touch the global frame — the isolation that makes programs modular.
5. Simultaneous assignment evaluates all right-side values before rebinding; use it for interdependent state updates.
6. Write tests immediately after (or before) implementing; use asserts, `_test.py` test functions, and doctests; exhaustive unit testing is a hallmark of good design.
7. Every loop pass must change a binding; statements after a `return` are never reached.

## Connects To
- **Section 1.2**: refines statements-executed vs expressions-evaluated, and extends the multiple-assignment rule into loop-state updates.
- **Section 1.4**: doctests live inside the docstring format defined there.
- **Section 1.6**: the `while`-based summation template (`sum_naturals`) is exactly the pattern higher-order functions will abstract.
- **Section 1.7**: iteration here is one repetition mechanism; recursion is the other.
- **Chapter 2**: every built-in data type has true and false values.
