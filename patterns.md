# Patterns & Techniques — Composing Programs

## Iterative Improvement
**When to use**: numerical problems solvable by refining a guess (square roots, golden ratio, Newton's method).
**How**: `improve(update, close, guess)` — loop `guess = update(guess)` until `close(guess)`.
**Trade-offs**: separates convergence logic from the specific computation; requires a good `close` tolerance.

## Extract the Pattern as a Higher-Order Function
**When to use**: two+ functions share a skeleton (summation over different terms, repeated transformation).
**How**: parameterize the varying part as a function argument (`summation(n, term)`); bind specifics with `lambda` or `def`.
**Trade-offs**: more abstraction layers to read; pays off from the second reuse.

## Closure for Persistent State
**When to use**: need stateful behavior without a class (counters, accounts, rate limiters).
**How**: outer function defines local state; inner function declares `nonlocal` and mutates; return the inner function.
**Trade-offs**: state is invisible from outside — good encapsulation, harder debugging.

## Constructor/Selector Data Abstraction
**When to use**: any compound value (pairs, rationals, intervals) before reaching for a class.
**How**: define `make_x(...)` + selectors; write all logic against selectors only; reduce to lowest terms (normalize) in the constructor.
**Trade-offs**: representation can be swapped freely; discipline is unenforced — barriers hold only if you respect them.

## Dispatch Dictionary
**When to use**: implementing message-based objects or generic operations without classes.
**How**: `{'deposit': deposit_fn, 'withdraw': withdraw_fn}`; a dispatch function routes message → handler.
**Trade-offs**: explicit and inspectable; no inheritance machinery.

## Type Dispatch / Coercion for Generic Functions
**When to use**: one operation over multiple types (complex + rational).
**How**: either branch on `type_tag` pairs (dispatch), or convert one type to the other and apply one implementation (coercion).
**Trade-offs**: dispatch grows quadratically in type pairs; coercion needs a sensible conversion hierarchy.

## Memoization
**When to use**: pure recursive functions recomputing the same subproblems (fib, path counting).
**How**: decorator holding a `cache` dict keyed by args; check before compute.
**Trade-offs**: Θ(φⁿ)→Θ(n) time for fib; costs memory; only valid for pure functions.

## Recursion Checklist
**When to use**: problems with natural self-similar decomposition.
**How**: (1) write base case(s) first; (2) decompose into smaller instance; (3) trust the recursive call (recursive leap of faith); (4) verify the combining step.
**Trade-offs**: tree recursion can be exponential — pair with memoization; Python has no tail-call optimization.

## Generator for Lazy Sequences
**When to use**: long, infinite, or partially consumed sequences; pipeline processing.
**How**: write a function with `yield`; iterate with `for`/`next`; compose generators into pipelines.
**Trade-offs**: one-pass only; state lives in suspended frames.

## Interpreter Skeleton (eval/apply)
**When to use**: a problem domain deserving its own language (calculators, config DSLs, query languages).
**How**: parser (tokenize → Pair/expression tree) + `eval` (dispatch on expression form) + `apply` (dispatch on operator), wrapped in a REPL with error handling.
**Trade-offs**: eval and apply are mutually recursive; start with combination only (Calculator), add abstraction (define, lambda) later.

## Exceptions as Non-Local Control
**When to use**: a deeply nested routine must signal to a distant caller (convergence failure, invalid input).
**How**: `raise` a specific exception class carrying data as attributes; `try/except` at the level that can respond.
**Trade-offs**: clearer than threading error codes through every return; overuse hides control flow.

## SQL Declarative Queries
**When to use**: data lives in relations; you need filtered, joined, aggregated views.
**How**: `SELECT ... FROM a, b WHERE join_condition` ; aggregates `MAX/COUNT/SUM` with `GROUP BY ... HAVING`.
**Trade-offs**: state *what*, not *how*; the interpreter chooses execution.

## Map-Reduce Decomposition
**When to use**: big-data processing parallelizable per-record.
**How**: `map` emits key-value pairs per record; framework groups by key; `reduce` combines each group.
**Trade-offs**: forces functional, stateless mappers — which is exactly what makes it distributable.
