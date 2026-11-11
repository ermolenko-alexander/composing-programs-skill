# Section 1.6: Higher-Order Functions

## Core Idea
Functions that manipulate functions — higher-order functions — let us name recurring *patterns* of computation (not just operations on numbers) and work in terms of those names directly, vastly increasing the expressive power of the language.

## Frameworks Introduced
- **Functions as arguments (the summation abstraction)**: when several functions share a common template differing only in one slot, turn the slot into a formal parameter.
  - When to use: `sum_naturals`, `sum_cubes`, `pi_sum` are "for the most part identical, differing only in name and the function of k" — such a common pattern is strong evidence of a useful abstraction waiting to be surfaced.
  - How: write the template, replace `<term>(k)` with a parameter `term`, pass `identity`, `cube`, or `pi_term`.
- **Iterative improvement** (functions as general methods): `improve(update, close, guess=1)` — begin with a guess, repeatedly apply an `update` function, and apply a `close` comparison to check whether the guess is close enough. The general method is independent of the particular functions it calls.
- **Lexical scoping** (nested definitions): inner functions have access to names in the environment where they are *defined*, not where they are called. Two extensions to the environment model: (1) each user-defined function has a *parent* environment — the environment in which it was defined; (2) when called, its local frame *extends* its parent environment. Environments become arbitrarily long chains of frames ending at the global frame.
- **Closures**: locally defined functions "enclose" information — `sqrt_update` carries the value of `a` from its defining environment; hence the name.
- **Newton's method**: an iterative improvement algorithm for finding *zeros* of differentiable functions: follow the tangent line — `update(x) = x - f(x)/df(x)` — until `f(x)` is near 0. Expressed as `find_zero(f, df) = improve(newton_update(f, df), near_zero)`.
- **Currying**: transform `f(x, y)` into `g` such that `g(x)(y) == f(x, y)`; automated by `curry2`/`uncurry2`. Useful when an interface requires a single-argument function (e.g. the *map pattern*, `map_to_range`).
- **Lambda expressions**: `lambda x: f(g(x))` evaluates to an unnamed function whose body is a single return expression; no assignment or control statements allowed. Read: "a function that takes x and returns f(g(x))".
- **First-class status**: elements with the fewest restrictions — they may be bound to names, passed as arguments, returned as results, and included in data structures. Python awards functions full first-class status.
- **Function decorators**: `@trace` above a `def` binds the name not to the defined function but to `trace(triple)` — the decorator applied to the newly defined function.

## Key Concepts
- **Higher-order function**: a function that accepts functions as arguments or returns functions as values.
- **Parent annotation**: a function value's parent is the first frame of the environment in which it was defined; the frame created by calling it has that same parent.
- **Extended environment**: a chain of frames (local → parent → … → global); name lookup follows the chain and takes the first binding found.
- **Function composition**: `compose1(f, g)` returns `h(x) = f(g(x))` — returned inner functions keep their parent environment.
- **Zero of a function**: an argument yielding return value 0; root-finding reduces many problems (nth roots: x such that x^n − a = 0) to zero-finding.
- **Lambda function**: result of a lambda expression; no intrinsic name (prints `<lambda>`), otherwise an ordinary function.
- **Map pattern**: applying a single-argument function to a sequence of values.
- **Decorator**: special `def` syntax applying a higher-order function to the newly defined function at definition time.

## Mental Models
- Use the "fill-in-the-template" test: if you could generate several functions by filling slots in one template, parameterize the slots.
- Think of a returned inner function as a package of code + defining environment (closure): it can run anywhere and still see `a`.
- Use nested definitions when (a) small helper names would clutter the global frame, or (b) an interface demands a fixed signature (e.g. `improve` needs a one-argument `update`) but your logic needs extra data — capture it lexically.
- Think of `lambda` as an expression-level `def` for simple functions passed as arguments or return values; prefer explicit `def` otherwise.

## Anti-patterns
- **Copy-pasting near-identical loop functions**: write `summation(n, term)` once instead.
- **Cluttering the global frame with tiny helper functions**: nest them where they are used.
- **Compound lambda expressions**: `compose1 = lambda f,g: lambda x: f(g(x))` is correct but "notoriously illegible" — Python style prefers `def`.
- **Maximal abstraction for its own sake**: expert programmers choose the level of abstraction appropriate to the task.
- **Assuming Newton's method always converges**: the initial guess must be sufficiently close to the zero and conditions on the function must hold.

## Code Examples
```python
def sqrt(a):
    def sqrt_update(x):
        return average(x, a/x)
    def sqrt_close(x):
        return approx_eq(x * x, a)
    return improve(sqrt_update, sqrt_close)
```
- **What it demonstrates**: nested definitions + lexical scoping — the inner functions read `a` from the parent frame, adapting a two-argument update to `improve`'s one-argument interface.

## Worked Example
Newton's method, built entirely from pieces already defined. `newton_update(f, df)` returns `update(x) = x - f(x)/df(x)` (follow the tangent to where it crosses 0). `find_zero(f, df)` defines `near_zero(x) = approx_eq(f(x), 0)` and returns `improve(newton_update(f, df), near_zero)`. Then `square_root_newton(a)` instantiates `f(x) = x*x - a`, `df(x) = 2*x` and calls `find_zero` → `square_root_newton(64)` is 8.0. Generalizing with `power`, `nth_root_of_a(n, a)` uses `f(x) = power(x, n) - a`, `df(x) = n * power(x, n-1)`: roots of degree 2, 3, 6 of 64 give 8.0, 4.0, 2.0. Three layers of higher-order functions (improve → find_zero → nth_root) each add generality; accuracy is tuned via `approx_eq`'s `tolerance`.

## Key Takeaways
1. A repeated template across functions is a higher-order abstraction waiting to be named (summation, improve).
2. Lexical scoping = two model rules: functions remember their defining environment (parent), and calls extend it.
3. Closures let you fit extra data through narrow interfaces without globals.
4. Currying converts multi-argument functions to single-argument chains for map-like interfaces; `curry2(f)(x)(y) == f(x, y)`.
5. Reserve lambda for simple functions used as arguments/return values; name everything else with `def`.
6. First-class functions mean general methods are ordinary values — bind, pass, return, store them.
7. `@decorator` is just `name = decorator(name)` after the `def`.

## Connects To
- **Section 1.3**: the environment model extends gracefully — no new mechanism beyond parent frames is needed for higher-order functions.
- **Section 1.5**: the while-loop summation and fib templates are what gets abstracted here; `improve_test` applies the testing discipline.
- **Section 1.7**: first-class functions plus self-reference enable recursion; lambda calculus returns in Chapter 3 interpreters.
