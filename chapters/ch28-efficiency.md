# Section 2.8: Efficiency

## Core Idea
Decisions about how to represent and process data are often driven by the efficiency of the alternatives. Efficiency means the computational resources a representation or process uses — how much time and memory it takes to compute a function's result or represent an object — and these amounts vary widely with implementation details. Since measuring exactly how long a program runs or how much memory it consumes depends on how a computer is configured, a more reliable way to characterize a program is to measure how many times some event occurs (such as a function call), and to classify processes by **orders of growth** — how resource requirements scale as a function of input size.

## Frameworks Introduced
- **Measuring by counting events** (2.8.1): wrap a function with a counter instead of timing it.
  - Time: higher-order `count(f)` returns an equivalent function carrying a `call_count` attribute. Tree-recursive `fib(19)` returns 4181 but makes 13,529 calls — the calls required grow faster than the Fibonacci numbers themselves, which is characteristic of tree-recursive functions.
  - Space: an environment is **active** if it provides the evaluation context for some expression currently being evaluated; it becomes inactive when the function call for which its first frame was created finally returns. The interpreter preserves all active environments and all values and frames referenced by them; inactive frames can be reclaimed. `count_frames(f)` tracks `open_count` (calls not yet returned) and `max_count` (maximum simultaneously active frames).
- **Memoization** (2.8.2): a memoized function stores the return value for any arguments it has previously received; a second call with the same arguments returns the existing value rather than recomputing. Expressed naturally as a higher-order function `memo(f)` (also usable as a decorator) with a dict `cache` indexed by arguments — which requires that the argument be immutable.
- **Orders of growth / Theta notation** (2.8.3): categorize a process with the group of processes that have similar requirements, glossing over machine-level constants.
- **Successive squaring** (2.8.4): restructure exponentiation so the problem halves at every even step — Θ(log n) time and space instead of Θ(n).
- **Growth categories and simplification rules** (2.8.5): constants, log bases, nesting, lower-order terms; a small set of common categories covers most processes.

## Evaluation Rules
The author's formal definitions, verbatim in substance:

**Theta notation.** Let n be a parameter that measures the size of the input to some process, and let R(n) be the amount of some resource the process requires for an input of size n (memory used, elementary machine steps performed, etc.; n could also be, e.g., the number of digits of accuracy required). We say R(n) has order of growth Θ(f(n)), written R(n) = Θ(f(n)) (pronounced "theta of f(n)"), if there are positive constants k1 and k2 **independent of n** such that

```
k1 · f(n) ≤ R(n) ≤ k2 · f(n)
```

for any value of n larger than some minimum m. For large n, R(n) is always sandwiched between two values that both scale with f(n): a lower bound k1·f(n) and an upper bound k2·f(n).

**Constants.** Constant terms never affect the order of growth: Θ(n) and Θ(500·n) are the same, because the definition lets us choose arbitrary constants k1 and k2 (such as 1/500). Constants are always omitted from orders of growth.

**Logarithms.** The base of a logarithm does not affect the order of growth: log₂n and log₁₀n are the same order, because changing the base is equivalent to multiplying by a constant factor.

**Nesting.** When an inner computational process is repeated for each step in an outer process, the order of growth of the entire process is the **product** of the number of steps in the outer and inner processes.

**Lower-order terms.** As input grows, the fastest-growing part of a computation dominates the total resources used. In a sum, all but the fastest-growing term can be dropped without changing the order of growth: Θ(n² + k·n) = Θ(n²) for any constant k, because n² eventually dominates — the requirement that bounds hold only for n greater than some minimum m is what establishes this equivalence. Lower-order terms are always omitted, so **a theta expression never contains a sum**.

**Space rule.** In evaluating an expression, the interpreter preserves all active environments and all values and frames referenced by them. An environment is active if it provides the evaluation context for some expression being evaluated, and becomes inactive when the call for which its first frame was created returns. For tree-recursive functions, space is proportional to the **maximum depth of the tree**: the interpreter only needs the nodes above the current node; memory for already-evaluated branches can be reclaimed because it cannot affect future computation.

## Code Listings (as the book gives them)
```python
def fib(n):
    if n == 0:
        return 0
    if n == 1:
        return 1
    return fib(n-2) + fib(n-1)

def count(f):
    def counted(*args):
        counted.call_count += 1
        return f(*args)
    counted.call_count = 0
    return counted

def count_frames(f):
    def counted(*args):
        counted.open_count += 1
        counted.max_count = max(counted.max_count, counted.open_count)
        result = f(*args)
        counted.open_count -= 1
        return result
    counted.open_count = 0
    counted.max_count = 0
    return counted

def memo(f):
    cache = {}
    def memoized(n):
        if n not in cache:
            cache[n] = f(n)
        return cache[n]
    return memoized

from math import sqrt
def count_factors(n):
    sqrt_n = sqrt(n)
    k, factors = 1, 0
    while k < sqrt_n:
        if n % k == 0:
            factors += 2
        k += 1
    if k * k == n:
        factors += 1
    return factors

def exp(b, n):
    if n == 0:
        return 1
    return b * exp(b, n-1)

def exp_iter(b, n):
    result = 1
    for _ in range(n):
        result = result * b
    return result

def square(x):
    return x*x

def fast_exp(b, n):
    if n == 0:
        return 1
    if n % 2 == 0:
        return square(fast_exp(b, n//2))
    else:
        return b * fast_exp(b, n-1)

def overlap(a, b):
    count = 0
    for item in a:
        if item in b:
            count += 1
    return count

def one_more(a):
    return overlap([x-1 for x in a], a)
```

## Worked Examples

### 1. Counting fib's calls, then memoizing it
Evaluating `fib(6)` evolves a tree: to compute `fib(5)` we compute `fib(3)` and `fib(4)`; to compute `fib(3)` we compute `fib(1)` and `fib(2)`. The function is instructive as a prototypical tree recursion but terribly inefficient — the entire computation of `fib(3)` is duplicated. Measure it:

```python
>>> fib = count(fib)
>>> fib(19)
4181
>>> fib.call_count
13529
```

13,529 calls to produce 4181 — calls grow faster than the Fibonacci numbers themselves. Now memoize, composing with `count` so we can still observe:

```python
>>> counted_fib = count(fib)
>>> fib = memo(counted_fib)
>>> fib(19)
4181
>>> counted_fib.call_count
20
>>> fib(34)
5702887
>>> counted_fib.call_count
35
```

With memoization, `fib` is called only **once per unique input**: `fib(19)` takes 20 calls, and extending all the way to `fib(34)` adds only 15 more. The new pattern of computation reuses the results for `fib(2)` and `fib(3)` when computing `fib(4)` on the right branch of the tree, so much of the tree-recursive computation is not required at all. (Note the rebinding pattern: the *recursive* calls inside the original `fib` must hit the memoized name, which is why `fib` itself is rebound.)

### 2. fib's space with count_frames
```python
>>> fib = count_frames(fib)
>>> fib(19)
4181
>>> fib.open_count
0
>>> fib.max_count
19
>>> fib(24)
46368
>>> fib.max_count
24
```
`max_count` — the maximum number of simultaneously active frames — is just 19 for `fib(19)` and 24 for `fib(24)`. In evaluating `fib(3)`, once `fib(n-2)` yields 0, its frame is no longer part of an active environment and a well-designed interpreter reclaims it; meanwhile the frame evaluating `fib(n-1)` and the original `fib(3)` frame (whose return value is not yet computed) stay active. Summary: the space requirement of `fib`, measured in active frames, is **one less than the input** — small; the time requirement in total recursive calls is **larger than the output** — huge.

### 3. A full Theta proof: count_factors is Θ(√n)
`count_factors` trial-divides n by every integer up to √n, exploiting that if k divides n and k < √n, there is another factor j = n/k with j > √n. The while body executes ⌊√n⌋ times; statements before and after execute once, giving roughly w·√n + v statements (w in the body, v outside). The "constants" w and v are not actually constant (the `factors` assignment sometimes runs, sometimes not) — order-of-growth analysis exists precisely to gloss over that.

The book's two-sided argument:
- **Lower bound.** Choose k1 = 1, m = 0. At least 4 lines execute outside the while, at least two within the body plus the header, each taking at least one step; the body runs at least √n − 1 times. So at least 4 + 3·(√n − 1) steps, always larger than k1·√n.
- **Upper bound.** Assume any single line requires at most p steps (not true of every line of Python, but true here). Then at most p·(5 + 4√n) steps — 5 lines outside, 4 within including the header, even if every `if` is true. Choosing k2 = 5p, steps are always smaller than k2·√n. Argument complete: Θ(√n).

### 4. Exponentiation: linear → constant space → logarithmic time
`exp(b, n)` implements bⁿ = b·bⁿ⁻¹, b⁰ = 1: a linear recursive process, Θ(n) steps and Θ(n) space. `exp_iter` is the equivalent linear iteration: similar steps, **constant space**. Successive squaring does better: b⁸ needs not seven multiplications but three (b² = b·b, b⁴ = b²·b², b⁸ = b⁴·b⁴), generalized by

```
b^n = (b^(n/2))²   if n is even
b^n = b · b^(n-1)  if n is odd
```

`fast_exp(2, 100)` → 1267650600228229401496703205376. The process grows logarithmically in both space and steps: computing b^(2n) requires only **one more multiplication** than computing bⁿ, so the computable exponent roughly doubles with every multiplication allowed — multiplications grow as log₂n. The difference becomes striking as n grows: n = 1000 needs only 14 multiplications instead of 1000.

### 5. Nesting and lower-order terms: overlap and one_more
`overlap(a, b)` counts elements of `a` that also appear in `b`. The `in` operator on a list is Θ(n) (n = len(b)); it is applied Θ(m) times (m = len(a)): `item in b` is the inner process, `for item in a` the outer, so the total is Θ(m·n). `one_more(a)` adds a Θ(n) list comprehension to a Θ(n²) overlap call; the sum Θ(n + n²) is not the simplest form — the n² term dominates, so it is Θ(n²).

## Reference Table — Common Categories
From slowest to fastest growth (the book's table, reproduced exactly):

| Category | Theta Notation | Growth Description | Example |
|---|---|---|---|
| Constant | Θ(1) | Growth is independent of the input | abs |
| Logarithmic | Θ(log n) | Multiplying input increments resources | fast_exp |
| Linear | Θ(n) | Incrementing input increments resources | exp |
| Quadratic | Θ(n²) | Incrementing input adds n resources | one_more |
| Exponential | Θ(bⁿ) | Incrementing input multiplies resources | fib |

Other categories exist — such as the Θ(√n) growth of `count_factors` — but these are particularly common. Exponential growth describes **many different** orders of growth, because changing the base b *does* affect the order. For fib specifically: the nth Fibonacci number is the closest integer to φ^(n−2)/√5, where φ = (1+√5)/2 ≈ 1.6180 (the golden ratio); since steps scale with the resulting value, tree-recursive fib requires Θ(φⁿ) steps.

## Key Concepts
- **Efficiency**: the computational resources (time, memory) used by a representation or process.
- **Counting events**: characterizing efficiency by how many times some event occurs (e.g. a function call) rather than by wall-clock measurement, which depends on machine configuration.
- **Active environment**: one providing the evaluation context for some expression being evaluated; becomes inactive when its originating call returns; only active environments (and what they reference) must be preserved.
- **call_count / open_count / max_count**: instrumentation attributes — total calls, calls not yet returned, and the max simultaneously active frames (the space measure).
- **Memoization**: caching return values keyed by previously received arguments; turns tree-recursive fib into one call per unique input.
- **Order of growth**: how a process's resource requirements grow as a function of input, within constant factors.
- **Theta notation Θ(f(n))**: two-sided bound k1·f(n) ≤ R(n) ≤ k2·f(n) for n > m.
- **Tree recursion cost asymmetry**: fib's time (calls) exceeds its output; its space (frames) is input − 1.
- **Successive squaring**: the even/odd recursive rule giving Θ(log n) exponentiation.
- **Nesting rule**: total growth = product of outer steps × inner steps.
- **Lower-order terms**: dropped; no sums inside Θ.
- **Golden ratio φ**: ≈ 1.618; base of fib's exponential growth.

## Mental Models
- Memoization trades a dictionary's space for exponential→linear time; it helps only when the same arguments recur (overlapping subproblems).
- To estimate the space of a recursive process, ask "how deep is the deepest chain of unreturned calls?" — never "how many calls total?". For tree recursion that's the maximum tree depth.
- Classify a loop nest by multiplying: steps(outer) × steps(inner body).
- Use the table's "growth description" column as a diagnostic: if *multiplying* the input adds a constant amount of work, it's logarithmic; if *incrementing* the input multiplies the work, it's exponential.
- Recognizing an input-halving structure (fast_exp on even n) signals Θ(log n): each unit of extra work doubles the solvable problem size.
- A Theta claim is a proof obligation in both directions: exhibit a k1/m pair for the lower bound and a k2 for the upper, as the count_factors argument does.

## Common Errors
Mistakes the author explicitly flags:
- **Measuring wall-clock time or raw memory** to characterize a program: results depend on many details of machine configuration; count events instead.
- **Seeking an exact step count**: "constants" like w and v in count_factors are not constant at all (conditional statements execute sometimes); order of growth exists to gloss over this.
- **Memoizing with mutable arguments**: the dict cache requires the memoized function's argument to be immutable.
- **Assuming every line of Python takes bounded time**: the upper-bound argument's "any single line requires at most p steps" assumption *isn't true for every line of Python* (e.g. `item in b` is Θ(n)); it must be checked per function.
- **Keeping constants, log bases, or lower-order terms inside Θ**: all are always omitted; a sum never appears within a theta expression.
- **Treating "exponential" as one class**: changing the base b *does* change the order of growth of Θ(bⁿ), unlike logarithms.
- **Leaving tree recursion with duplicated subcomputations unmemoized**: the entire computation of fib(3) is redone inside fib(6); this redundancy is the defining inefficiency of naive tree recursion.

## Anti-patterns
- Timing instead of counting for analysis.
- Linear-scan membership (`in` on a list) inside a loop over another list without noticing the Θ(m·n) product.
- Writing linear-recursive accumulation (like `exp`) when an iteration gives the same time in constant space.
- Micro-optimizing constants when a structural change (memoization, successive squaring) changes the growth category.

## Key Takeaways
1. Characterize efficiency by counting events — total calls for time, maximum simultaneously active frames for space — not wall-clock measurements.
2. fib shows the canonical asymmetry: space n−1 (small), time 13,529 calls for fib(19) (huge, growing faster than the output).
3. Memoization makes a tree-recursive function call its underlying computation once per unique input: fib(19) in 20 calls, fib(34) in 35.
4. Θ(f(n)) is a two-sided sandwich with constants k1, k2 independent of n, holding beyond some minimum m; proving it means exhibiting both bounds.
5. Simplify by rule: drop constants and log bases, multiply nested processes, drop lower-order terms — never a sum inside Θ.
6. Know the five common categories (constant, logarithmic, linear, quadratic, exponential) and their increment-effect intuitions; place new processes among them, remembering extras like Θ(√n).
7. Structural insight beats micro-optimization: successive squaring turns 1000 multiplications into 14.

## Connects To
- **Section 1.7 (tree recursion)**: fib is "our first tree-recursive function"; this section finally quantifies its cost.
- **Section 1.6 (higher-order functions / decorators)**: count, count_frames, and memo are all higher-order functions; memo can be used as a decorator.
- **Section 2.4 (dictionaries, mutable data)**: memo's cache is a dict with local state captured in a closure; dict keys must be immutable.
- **Section 2.9 (recursive objects)**: Θ analysis carries forward to set operations — linked-list sets Θ(n)/Θ(n²), sorted lists, and Θ(log n) binary search trees.
- **Environment model (1.x)**: the active-frame space rule is stated entirely in terms of the environment model of computation.
