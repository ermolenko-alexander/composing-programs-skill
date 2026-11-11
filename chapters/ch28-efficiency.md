# Section 2.8: Efficiency

## Core Idea
Efficiency is the computational resources (time, memory) a representation or process uses. Since exact timing depends on machine details, characterize programs by counting events (calls, active frames) and by **orders of growth** — how resource requirements scale as a function of input size.

## Frameworks Introduced
- **Measuring by counting events**: wrap a function with a counter rather than timing it.
  - Time: `count(f)` returns a function with a `call_count` attribute — tree-recursive `fib(19)` makes 13,529 calls to produce 4,181.
  - Space: active environments only — an environment is **active** if it provides the evaluation context for some expression being evaluated, and becomes inactive when its call returns; `count_frames(f)` tracks `open_count`/`max_count` (max simultaneously active frames). For `fib(n)` max_count is n — space proportional to the maximum depth of the tree.
- **Memoization**: store the return value for any arguments previously received; a second call returns the cached value.
  - When to use: tree-recursive processes that repeat computation.
  - How: higher-order `memo(f)` with a dict `cache` indexed by arguments (arguments must be immutable); memoized fib is called once per unique input (`fib(34)` → 35 calls instead of millions).
- **Orders of growth / Theta notation**: `R(n) = Θ(f(n))` iff there exist positive constants k1, k2 independent of n with `k1·f(n) ≤ R(n) ≤ k2·f(n)` for all n larger than some minimum m — R is sandwiched between a lower and an upper bound that both scale with f(n).
- **Simplification rules**:
  - **Constants**: never affect order of growth (Θ(500n) = Θ(n)).
  - **Logarithms**: base doesn't matter (changing base = constant factor).
  - **Nesting**: inner process repeated for each outer step → total = product of inner and outer step counts.
  - **Lower-order terms**: drop everything but the fastest-growing term (Θ(n² + kn) = Θ(n²)); never a sum inside a theta expression.
- **Successive squaring (fast_exp)**: `b^n = (b^(n/2))² if n even, else b·b^(n-1)` — Θ(log n) time and space vs Θ(n); each extra multiplication doubles the computable exponent (n=1000 needs 14 multiplications, not 1000).

## Key Concepts
- **Tree recursion cost**: naive `fib` duplicates whole subtrees (fib(3) computed repeatedly); calls grow faster than the Fibonacci numbers themselves — Θ(φⁿ) steps, φ = (1+√5)/2 ≈ 1.618 (the nth Fibonacci number ≈ φ^(n-2)/√5).
- **Space vs time asymmetry of fib**: space (active frames) is n−1, small; time (calls) is huge.
- **count_factors**: counts divisors by trial division up to √n (each divisor k < √n pairs with j = n/k > √n) — Θ(√n) time; used to demonstrate a full lower/upper-bound Theta proof.
- **exp vs exp_iter**: linear recursion Θ(n) time and Θ(n) space; iteration same time, constant space.
- **overlap(a, b)**: `in` on a list is Θ(n), applied Θ(m) times → Θ(m·n); `one_more` = Θ(n) comprehension + Θ(n²) overlap = Θ(n²).

## Mental Models
- Think of memoization as trading a dictionary's space for exponential→linear time; it only helps when the same arguments recur.
- Classify a loop nest by multiplying: steps(outer) × steps(inner body).
- To estimate space of a recursive process, ask "how deep is the deepest chain of unreturned calls?", not "how many calls total?".
- Recognizing an exponent-halving structure (fast_exp, binary search–like) signals Θ(log n).

## Anti-patterns
- **Timing instead of counting** for analysis: results depend on machine configuration; count events.
- **Leaving tree recursion unmemoized** when subproblems repeat.
- **Keeping constants and lower-order terms in Θ**: they are always omitted.
- **Memoizing functions of mutable arguments**: dict cache requires immutable keys.
- **Treating all exponentials as one class**: changing the base b *does* change the order of growth of Θ(bⁿ).

## Code Examples
```python
def memo(f):
    cache = {}
    def memoized(n):
        if n not in cache:
            cache[n] = f(n)
        return cache[n]
    return memoized

def fast_exp(b, n):
    if n == 0:
        return 1
    if n % 2 == 0:
        return square(fast_exp(b, n//2))
    else:
        return b * fast_exp(b, n-1)
```
- **What it demonstrates**: the two signature optimizations of the section — caching repeated subproblems and halving the problem size.

## Reference Tables
| Category | Theta Notation | Growth Description | Example |
|---|---|---|---|
| Constant | Θ(1) | Growth is independent of the input | abs |
| Logarithmic | Θ(log n) | Multiplying input increments resources | fast_exp |
| Linear | Θ(n) | Incrementing input increments resources | exp |
| Quadratic | Θ(n²) | Incrementing input adds n resources | one_more |
| Exponential | Θ(bⁿ) | Incrementing input multiplies resources | fib |

(Other categories exist, e.g. Θ(√n) for count_factors.)

## Worked Example
Memoized Fibonacci. Instrument naive `fib` with `count`: `fib(19)` = 4181 takes 13,529 calls — the tree duplicates entire subcomputations. Wrap with `count_frames`: `max_count` is only 19 — just one root-to-leaf chain is ever active, so space is linear while time is exponential. Now compose `fib = memo(count(fib))`: computing `fib(19)` makes 20 calls, and extending to `fib(34)` = 5,702,887 adds only 15 more (35 total) — each unique argument computed once, the right branch of the tree reusing the left's cached results. The same analysis vocabulary then formalizes as Θ: naive fib Θ(φⁿ) time / Θ(n) space, memoized fib Θ(n) time.

## Key Takeaways
1. Characterize efficiency by counting events (calls, max active frames), not wall-clock time.
2. Memoize tree recursion with repeated subproblems: exponential time collapses to one call per unique argument.
3. Space = maximum simultaneously active environments; inactive frames are reclaimable.
4. State costs in Θ: drop constants, log bases, and lower-order terms; multiply nested processes.
5. Know the five common categories and their "growth description" intuitions; place any new process among them.
6. Structural insight beats micro-optimization: successive squaring turns Θ(n) into Θ(log n).

## Connects To
- **Section 2.3 / 1.x tree recursion**: fib and partition-style recursion are the processes being measured.
- **Section 2.9**: Θ analysis applies to set operations (linked-list sets Θ(n)/Θ(n²), sorted lists, and Θ(log n) binary search trees).
- **Section 2.4**: the functional dictionary's linear-scan lookup is exactly the inefficiency Θ notation names.
