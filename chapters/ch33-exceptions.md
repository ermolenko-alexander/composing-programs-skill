# Section 3.3: Exceptions

## Core Idea
Exceptions are a general mechanism for interrupting the normal flow of execution, signaling that an exceptional circumstance has arisen, and returning control directly to an enclosing part of the program designated to react to it — separating main logic from error-handling logic as modular parts. Programmers must make conscious choices about how programs react to exceptional conditions: a web server logs errors and keeps serving requests; the Python interpreter terminates immediately and prints a message so issues get fixed at once. Neither policy is universally right.

## Frameworks Introduced
- **raise / try-except control transfer**: raising aborts the current block; control jumps across function-call boundaries to the handler.
  - When to use: any exceptional condition (bad arguments, missing resource, lost connection) where the reaction belongs elsewhere than the detecting code.
- **Exceptions as data-carrying signals**: a user-defined exception class can hold attributes, turning the exception path into a channel for partial results.
  - When to use: an algorithm fails mid-way but has useful intermediate state (the best guess so far).
- **Error-handling policy is a design choice**: robust-and-logging vs fail-fast, chosen per program.

## Evaluation Rules (the author's formal procedures)

**Raising.** An exception is an object instance with a class that inherits, directly or indirectly, from `BaseException`. `assert` raises an `AssertionError`; any exception instance can be raised with `raise`, most commonly by constructing it inline:
```python
>>> raise Exception('An error occurred')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
Exception: an error occurred
```
When an exception is raised, no further statements in the current block execute. Unless handled, the interpreter returns directly to the interactive read-eval-print loop, or terminates entirely if Python was started with a file argument, printing a **stack backtrace** — structured text describing the nested set of active function calls in the branch of execution where the exception was raised (`<stdin>` as filename means it was raised interactively).

**Handling.** A `try` statement consists of multiple clauses:
```python
try:
    <try suite>
except <exception class> as <name>:
    <except suite>
...
```
Rules, verbatim-faithfully:
1. The `<try suite>` is always executed immediately when the `try` statement is executed.
2. Suites of `except` clauses are executed only when an exception is raised during the course of executing the `<try suite>`.
3. Each `except` clause specifies the particular class of exception to handle: if the class is `AssertionError`, any instance of a class inheriting from `AssertionError` raised during the try suite is handled by that except suite (subclass matching).
4. Within the `<except suite>`, `<name>` is bound to the raised exception object — but this binding does not persist beyond the except suite.
5. A `try` statement handles exceptions that occur within the body of a function applied (directly or indirectly) within the try suite. When an exception is raised, control jumps directly to the body of the `<except suite>` of the **most recent** `try` statement that handles that type of exception.

## Key Concepts
- **Exception**: object instance whose class inherits from `BaseException`.
- **Raising**: interrupting normal flow; done by `raise`, `assert`, and by the interpreter itself on every detected error.
- **Handling**: catching a raised exception with an enclosing `try` statement's matching `except` clause.
- **Stack backtrace**: printed description of the nested active calls at the raise point.
- **`as <name>` binding**: scoped to the except suite only.
- **Subclass matching**: `except C` handles instances of `C` and anything inheriting from it.
- **Unhandled exception**: control returns to the REPL, or Python terminates when run on a file.
- **Exception attributes**: exceptions carry data — the assert message, raise-location info, and any attributes a user-defined class adds.

## Worked Examples

### 1. Handling ZeroDivisionError locally
```python
>>> try:
        x = 1/0
    except ZeroDivisionError as e:
        print('handling a', type(e))
        x = 0
handling a <class 'ZeroDivisionError'>
>>> x
0
```
The try suite starts, `1/0` raises, the assignment never completes, control lands in the except suite, which binds `e`, prints, and sets `x = 0`.

### 2. Control jumping across call boundaries
```python
>>> def invert(x):
        result = 1/x  # Raises a ZeroDivisionError if x is 0
        print('Never printed if x is 0')
        return result

>>> def invert_safe(x):
        try:
            return invert(x)
        except ZeroDivisionError as e:
            return str(e)

>>> invert_safe(2)
Never printed if x is 0
0.5
>>> invert_safe(0)
'division by zero'
```
With `x=0`, the `print` in `invert` is never evaluated: the raise inside `invert` transfers control straight to the except clause in `invert_safe`, one frame up. Coercing the error to a string gives the human-readable `'division by zero'`.

### 3. IterImproveError for Newton's method (3.3.1 Exception Objects)
Chapter 1's Newton's method fails outright when probing raises an error. The fix: a domain exception that carries the best guess.

```python
>>> class IterImproveError(Exception):
        def __init__(self, last_guess):
            self.last_guess = last_guess

>>> def improve(update, done, guess=1, max_updates=1000):
        k = 0
        try:
            while not done(guess) and k < max_updates:
                guess = update(guess)
                k = k + 1
            return guess
        except ValueError:
            raise IterImproveError(guess)

>>> def find_zero(f, guess=1):
        def done(x):
            return f(x) == 0
        try:
            return improve(newton_update(f), done, guess)
        except IterImproveError as e:
            return e.last_guess
```
Walkthrough on f(x) = 2x² + √x, which has a zero at 0 but raises a math domain error (a type of `ValueError`) whenever `sqrt` is applied to a negative number:
1. `find_zero` calls `improve` with Newton's update (`newton_update(f)` from Chapter 1, unchanged).
2. Newton iteration proceeds inside `improve`'s try suite until an update probes a negative x; `sqrt` raises `ValueError`.
3. `improve`'s except clause fires and raises `IterImproveError(guess)` — the low-level error translated into a domain exception storing the most recent guess as an attribute.
4. That exception propagates up to `find_zero`'s handler, which returns `e.last_guess`.
```python
>>> from math import sqrt
>>> find_zero(lambda x: 2*x*x + sqrt(x))
-0.030211203830201594
```
Still far from the correct answer of 0, but some applications prefer this coarse approximation to a `ValueError`. Note the layering: iterative-improvement logic sits unchanged in the try clause; error logic sits in except clauses — exceptions as a separation-of-concerns tool.

## Mental Models
- `raise` is a teleport: no further statements in the current block run; control lands in the nearest matching except suite up the call stack.
- Exceptions are a modularity tool, not just error reporting: the try suite keeps the clean algorithm, the except clauses keep the contingency logic.
- Custom exception classes are typed, data-carrying signals between distant layers of a program.
- Two policies, choose consciously: log-and-continue (long-running services) vs terminate-and-report (interactive development).

## Common Errors / Anti-patterns
- **One error policy for all programs**: servers should log-and-continue; dev tools should fail fast.
- **Expecting `as e` to persist**: the name is unbound after the except suite — copy out what you need.
- **Losing intermediate results on failure**: Chapter 1's Newton's method raised ValueError and returned nothing; wrap and re-raise with the best guess instead.
- **Catching too broadly**: matching by class is the mechanism for precision — catch the specific class whose meaning you understand at this layer (e.g. `ValueError` in `improve`, `IterImproveError` in `find_zero`), and translate upward rather than swallowing.
- **Handling where you can't act**: the handler belongs at the layer that knows what the error means and what to do about it.

## Key Takeaways
1. An exception is just an instance of a `BaseException` subclass; raising it jumps to the most recent enclosing `try` that handles its type, even across many stack frames.
2. The try suite always runs; except suites run only on a matching raise; `as`-bindings are suite-local.
3. Subclass `Exception` and attach attributes to carry partial results or context across the jump.
4. Wrap low-level exceptions into domain-level ones at the layer that knows what they mean (`ValueError` → `IterImproveError`).
5. Exceptions keep algorithm and error handling in separate, modular suites — and they are the machinery interpreters use to stay robust.

## Connects To
- **Section 3.4**: the Calculator REPL's `try` reports `SyntaxError`, `TypeError`, `ValueError`, `ZeroDivisionError` without exiting, and catches `KeyboardInterrupt`/`EOFError` to quit cleanly; `calc_apply` raises `TypeError` for bad argument counts.
- **Section 3.1**: exceptions are the Python feature that makes interpreter implementations robust.
- **Chapter 1 (1.6)**: `improve` and `newton_update` — the generic iterative-improvement framework being hardened here.
