# Section 1.7: Recursive Functions

## Core Idea
A function is recursive if its body calls the function itself, directly or indirectly; recursion needs no special syntax or new evaluation rules — the environment model already explains it — and it expresses computation by simplifying problems incrementally.

## Frameworks Introduced
- **The anatomy of recursive functions**: the body begins with a *base case* — a conditional defining behavior for the simplest inputs (there may be several) — followed by one or more *recursive calls*, which always simplify the original problem so there is less work left for each subsequent call.
  - When to use: designing any recursive function.
  - How: find a way to break the problem into simpler problems of the same kind (e.g. `%`/`//` split a number into last digit and all but last digit); handle the simplest inputs directly.
- **The recursive leap of faith**: treat a recursive call as a functional abstraction — do not care how `fact(n-1)` is implemented; trust it computes (n−1)! and only check that n! is correct under that assumption. Verifying a recursive function this way is a form of proof by induction.
- **Iteration vs recursion (state location)**: iterative functions must maintain local state (`total`, `k`) characterizing completed and remaining work; in a recursive function the state of the computation is entirely contained within the structure of the environment — return values take the role of `total`, and `n` is bound to different values in different frames.
- **Mutual recursion**: a recursive procedure divided among two functions that call each other (`is_even`/`is_odd`); no more powerful than simple recursion (the bodies can be merged by breaking the abstraction boundary), but it maintains abstraction within a complicated recursive program — e.g. each player's strategy in its own function.
- **Tree recursion**: a function that calls itself more than once; each call branches into multiple smaller calls like branches of a tree. Think of it as *exploring different possibilities* — each recursive call corresponds to one possibility.

## Key Concepts
- **Base case**: the conditional branch handling inputs simplest to process (single digit, n == 1, n == 0).
- **Recursive call**: a self-application on a simpler instance of the same problem.
- **Circularity resolved by `def` semantics**: the body is not executed when the `def` runs, so a self-reference in the body is harmless; each application gets its own frame with a different argument.
- **Unwinding**: the result is built from the base case upward as calls return; each call's result depends on the next.
- **Mutually recursive**: two (or more) functions whose recursion flows through each other.
- **Tree recursive**: multiple direct recursive calls in one body (`fib(n-2) + fib(n-1)`).
- **Partition**: a way to express n as a sum of positive integer parts up to m in increasing order.

## Mental Models
- Use the recursive leap of faith when writing or reviewing: assume the recursive call is correct for the simpler input and verify only the combining step and base cases.
- Think of each recursive application as a fresh frame: `sum_digits` is "applied twice, but with a different argument each time" — no new model rules needed.
- Use printing around a recursive call (`cascade`) to visualize the process: output before the call happens on the way down, output after it on the way up.
- Choose tree recursion when the problem naturally splits into "with this choice / without this choice" possibilities (partitions: use a part of size m, or don't).
- Prefer recursion when correctly maintaining iterative local state would be the hard part; prefer iteration when efficiency matters (tree-recursive `fib` is far less efficient than the iterative version).

## Anti-patterns
- **Recursive calls that don't simplify**: every call must move toward a base case, or the recursion never terminates.
- **Missing base cases**: `count_partitions` needs three (n == 0 → 1; n < 0 → 0; m == 0 → 0) — forgetting the negative or zero-part cases breaks it.
- **Unwinding mentally instead of trusting the abstraction**: tracing every frame is possible but it is "often clearer to think about recursive calls as functional abstractions".
- **Assuming base cases must come textually first**: `cascade` can print before the conditional; it's not a rigid requirement.
- **Using tree recursion where iteration is easy**: the iterative `fib` is more efficient; reserve tree recursion for problems (like partitions) where the recursive solution is substantially simpler than any iterative alternative.

## Code Examples
```python
def count_partitions(n, m):
    """Count the ways to partition n using parts up to m."""
    if n == 0:
        return 1
    elif n < 0:
        return 0
    elif m == 0:
        return 0
    else:
        return count_partitions(n-m, m) + count_partitions(n, m-1)
```
- **What it demonstrates**: tree recursion as exploration of possibilities — partitions that include at least one m vs those that don't — plus multiple base cases.

## Worked Example
`sum_digits(738)`: since 738 ≥ 10, split into `all_but_last = 738 // 10 = 73` and `last = 738 % 10 = 8`, and return `sum_digits(73) + 8`. A second frame binds n = 73 → split into 7 and 3 → `sum_digits(7) + 3`. A third frame has n = 7 < 10 (base case) → returns 7. The results unwind: 7 + 3 = 10 in the second frame, 10 + 8 = 18 in the first. Each successive call takes a smaller argument until a single-digit input is reached — a simple body evolving a complex computational process with no new evaluation rules.

For partitions, `count_partitions(6, 4)`: partitions of 6 with parts up to 4 split into those containing a 4 (each is a partition of 6−4 = 2 with parts up to 4) plus those without (partitions of 6 with parts up to 3); recursing yields 9.

## Key Takeaways
1. Structure every recursive function as base case(s) + simplifying recursive call(s).
2. Verify by the recursive leap of faith: assume the simpler case works, check the combination — induction, not frame-tracing.
3. Recursion stores computation state in the environment's frames; iteration stores it in explicit local names — recursion often avoids state-bookkeeping bugs.
4. Mutual recursion = simple recursion + preserved abstraction boundaries (one function per strategy/role).
5. Tree recursion explores possibilities; use it when the branching decomposition is natural, and expect efficiency costs (tree `fib`).
6. Enumerate all base cases explicitly, including degenerate ones (negative n, zero-sized parts).

## Connects To
- **Section 1.3**: recursion works because `def` stores the body unevaluated and each call creates an independent local frame.
- **Section 1.5**: `fact_iter` vs `fact` and the two `fib` implementations contrast iteration's explicit state with recursion's environmental state.
- **Section 1.2.5**: the evaluation procedure for call expressions was already recursive; recursive functions apply the same idea at the program level.
