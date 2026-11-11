# Section 3.3: Exceptions

## Core Idea
Exceptions are a general mechanism for interrupting normal execution and returning control directly to an enclosing part of the program designated to handle the circumstance — separating main logic from error-handling logic as modular parts.

## Frameworks Introduced
- **raise/try-except**: raising an exception aborts the current block; control jumps to "the most recent try statement that handles that type of exception" — even across function-call boundaries.
  - When to use: any exceptional condition (bad arguments, missing resource, lost connection) where the reaction belongs elsewhere than the detecting code.
  - How: `raise SomeException(...)` constructs and raises an instance; an enclosing `try: <try suite> / except <class> as <name>: <except suite>` handles any instance of a class inheriting from `<class>` raised during the try suite (directly or in functions it calls).
- **Exceptions as non-local return values (exception objects carry data)**: a user-defined exception class can hold attributes, turning the exception path into a channel for partial results.
  - When to use: an algorithm fails mid-way but has useful intermediate state (e.g. the best guess so far).
  - How: subclass `Exception`, store data in `__init__`, raise; the handler reads the attribute (`e.last_guess`).
- **Error-handling policy is a design choice**: a persistent service (web server) logs and continues; an interactive interpreter terminates immediately with a message — "no single correct approach".

## Key Concepts
- **Exception**: an object instance whose class inherits (directly or indirectly) from `BaseException`.
- **Raising**: interrupting normal flow to signal an exceptional circumstance; `raise` and `assert` do it; the interpreter raises on every detected error.
- **Handling**: catching a raised exception with an enclosing `try` statement's `except` clause.
- **Stack backtrace**: printed structured text describing the nested active calls where the exception arose.
- **`as <name>` binding**: binds the exception object within the except suite only; the binding does not persist beyond it.
- **Exception class matching**: an `except AssertionError` clause handles instances of AssertionError and any subclass.
- **Unhandled exception**: returns control to the interactive REPL, or terminates Python run on a file.

## Mental Models
- Think of `raise` as a teleport: no further statements in the current block run; control lands in the nearest matching except suite up the call stack.
- Think of exceptions as a modularity tool, not just error reporting: the try suite keeps the clean algorithm, the except clauses keep the contingency logic.
- Use custom exception classes as typed, data-carrying signals between distant layers of a program.

## Anti-patterns
- **Letting one error policy fit all programs**: servers should log-and-continue; dev tools should fail fast — choose consciously.
- **Expecting `as e` to persist**: the name is unbound after the except suite.
- **Losing intermediate results on failure**: Chapter 1's Newton's method raised ValueError and returned nothing; wrap and re-raise with the best guess instead.

## Code Examples
```python
class IterImproveError(Exception):
    def __init__(self, last_guess):
        self.last_guess = last_guess

def improve(update, done, guess=1, max_updates=1000):
    k = 0
    try:
        while not done(guess) and k < max_updates:
            guess = update(guess)
            k = k + 1
        return guess
    except ValueError:
        raise IterImproveError(guess)

def find_zero(f, guess=1):
    def done(x):
        return f(x) == 0
    try:
        return improve(newton_update(f), done, guess)
    except IterImproveError as e:
        return e.last_guess
```
- **What it demonstrates**: translating a low-level ValueError into a domain exception carrying the last guess, caught at the layer that can use it.

## Worked Example
Finding a zero of 2x² + √x with Newton's method: the function has a zero at 0, but any negative probe makes `sqrt` raise a math-domain ValueError, so the Chapter 1 implementation dies with no answer. Revised design: `improve` catches ValueError and raises `IterImproveError(guess)`, preserving the most recent guess; `find_zero` catches `IterImproveError` and returns `e.last_guess`. Result: `find_zero(lambda x: 2*x*x + sqrt(x))` returns ≈ −0.0302 — a coarse approximation many applications prefer to a crash.

## Key Takeaways
1. Decide your error policy per program: robust-and-logging vs fail-fast are both valid.
2. `try` handles exceptions raised anywhere inside functions called from the try suite — handlers can live far from raisers.
3. Subclass `Exception` and attach attributes to carry partial results or context across the jump.
4. Wrap low-level exceptions into domain-level ones at the layer that knows what they mean.
5. Exceptions keep algorithm and error handling in separate, modular suites — and they are the machinery interpreters use for their own error reporting.

## Connects To
- **Section 3.4**: the Calculator REPL uses try/except to report SyntaxError, TypeError, ValueError, ZeroDivisionError without exiting, and KeyboardInterrupt/EOFError to quit cleanly.
- **Section 3.1**: exceptions are the Python feature that makes interpreter implementations robust.
