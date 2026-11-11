# Section 1.7: Recursive Functions

## Core Idea
"A function is called *recursive* if the body of the function calls the function itself, either directly or indirectly" — executing the body may require applying the function again. Recursive functions use no special syntax in Python and require **no new evaluation rules**: the environment model already explains them. They "express computation by simplifying problems incrementally," and functions with simple bodies can evolve complex computational processes through recursion.

## Frameworks Introduced

### The Anatomy of Recursive Functions
A common pattern in the body of many recursive functions:
- The body begins with a **base case** — a conditional statement defining the behavior for the inputs that are *simplest to process* (for `sum_digits`, any single-digit argument, returned as-is). Some recursive functions have multiple base cases.
- Base cases are followed by one or more **recursive calls**, which "always have a certain character: they simplify the original problem." Summing the digits of 7 is simpler than of 73, which is simpler than of 738 — for each subsequent call, there is less work left to be done.
- Design approach: look for ways a problem breaks into *simpler problems of the same kind*. For digit sums, `%` and `//` split a number into its last digit (`18117 % 10` → 7) and all but its last digit (`18117 // 10` → 1811); the recursion is complete once the one-digit special case is handled.

### The Recursive Leap of Faith
We *can* unwind a recursion with the model of computation, "but it is often clearer to think about recursive calls as functional abstractions." Do not care how `fact(n-1)` is implemented in the body of `fact`; "simply trust that it computes the factorial of n-1" — then only check that `n!` is computed correctly under that assumption (n! = n · (n−1)! by the mathematical definition). "Verifying the correctness of a recursive function is a form of proof by induction."

### Iteration vs Recursion — where state lives
- `fact_iter` must introduce two additional names, `total` and `k`, not required recursively. "In general, iterative functions must maintain some local state that changes throughout the course of computation. At any point in the iteration, that state characterizes the result of completed work and the amount of work remaining" (k=3, total=2 ⇒ terms 3 and 4 remain).
- `fact` is characterized by its single argument `n`: "The state of the computation is entirely contained within the structure of the environment," with return values taking the role of `total` and `n` bound to different values in different frames rather than explicitly tracking `k`.
- Consequence: "Recursive functions leverage the rules of evaluating call expressions to bind names to values, often avoiding the nuisance of correctly assigning local names during iteration. For this reason, recursive functions can be easier to define correctly" — though recognizing the processes they evolve requires practice.
- Conceptual contrast: the iterative factorial builds from the base case of 1 up to the final total; the recursive one constructs the result directly from the final term `n` and the result of the simpler problem `fact(n-1)`, with the result built starting from the base case as the recursion "unwinds."

### Mutual Recursion
A recursive procedure divided among two functions that call each other. Even/odd definition for non-negative integers: a number is even if it is one more than an odd number; odd if one more than an even number; 0 is even. Mutually recursive functions can always be turned into a single recursive function by *breaking the abstraction boundary* (inlining `is_odd` into `is_even`, replacing `n` with `n-1`). So "mutual recursion is no more mysterious or powerful than simple recursion, and it provides a mechanism for maintaining abstraction within a complicated recursive program" — e.g., one function per player's strategy in the pebble game, so a strategy can change without touching the other.

### Printing in Recursive Functions
The computational process can be visualized with `print` calls (`cascade`). Output placed *before* the recursive call appears on the way down; output *after* it appears on the way up. Also: "It is not a rigid requirement that base cases be expressed before recursive calls" — `cascade` compacts by hoisting the repeated `print(n)` above the conditional.

### Tree Recursion
"Another common pattern of computation … in which a function calls itself more than once." Each call branches into multiple smaller calls "just as the branches of a tree become smaller but more numerous as they extend from the trunk." The tree-recursive `fib` is "tremendously appealing" because it exactly mirrors the familiar definition — but the earlier iterative attempts were *more efficient* (a topic for later in the text). Mental model: "We can think of a tree-recursive function as exploring different possibilities" — each recursive call corresponds to one possibility (partitions: use a part of size m, or don't).

## Evaluation Rules (why recursion needs no new machinery)
- **def-time circularity is harmless**: when the `def` statement executes, `sum_digits` is bound to a new function, "but the body of that function is not yet executed. Therefore, the circular nature of sum_digits is not a problem yet."
- **Each application is independent**: the recursive function "applies correctly, despite its circular character, because it is applied twice, but with a different argument each time" — each application creates its own local frame binding `n` to its own value, and the second application is a *simpler instance* of the problem.
- **Unwinding**: the recursion ends by passing the base-case argument; "the result of each call depends on the next until the base case is reached," then results propagate back up frame by frame.

## Key Concepts
- **Recursive function**: body calls itself directly or indirectly.
- **Base case**: conditional branch for the simplest inputs (`n < 10`; `n == 1`; `n == 0`); possibly several.
- **Recursive call**: a self-application on a simplified instance.
- **Recursive leap of faith**: treat the recursive call as a functional abstraction; verification = induction.
- **Unwinding**: building the result from the base case back up through the frames.
- **Mutually recursive**: functions whose recursion flows through each other (`is_even`/`is_odd`, `play_alice`/`play_bob`).
- **Tree recursive**: multiple direct recursive calls in one body (`fib(n-2) + fib(n-1)`).
- **Partition**: a way of expressing n as a sum of positive integer parts up to m in increasing order.
- **State in the environment**: recursion's replacement for iteration's explicit local names.

## Mental Models
- Write/review with the leap of faith: assume the recursive call is correct for the simpler input; verify only the combining step and the base cases.
- Each recursive application = a fresh frame with a smaller argument; generate the diagram for `sum_digits(18117)` to watch arguments shrink to a single digit.
- `print` before the call = on the way down; after = on the way up (`cascade`'s palindrome output).
- Tree recursion = exploring possibilities; find the "with this choice / without it" split.
- Prefer recursion when correctly maintaining iterative state is the hard part; prefer iteration when efficiency matters (tree `fib` is far less efficient).

## Anti-patterns
- **Recursive calls that don't simplify**: every call must move toward a base case or recursion never terminates.
- **Missing base cases**: `count_partitions` needs three — partitions of 0 (exactly one: include no parts), negative n (zero ways), m == 0 with n > 0 (zero ways). Forgetting the degenerate cases breaks it.
- **Frame-tracing instead of trusting the abstraction**: possible but usually less clear than the leap of faith.
- **Assuming base cases must come textually first**: `cascade`'s compact form prints before the conditional.
- **Tree recursion where iteration is easy**: the iterative `fib` is more efficient; reserve tree recursion for problems like partitions, where "implementing this function without recursion would be substantially more involved."
- **Breaking abstraction boundaries unnecessarily**: inlining `is_odd` into `is_even` works but sacrifices the modularity mutual recursion preserves.

## Code Examples
```python
def sum_digits(n):
    """Return the sum of the digits of positive integer n."""
    if n < 10:
        return n
    else:
        all_but_last, last = n // 10, n % 10
        return sum_digits(all_but_last) + last
# sum_digits(9) -> 9; sum_digits(18117) -> 18; sum_digits(9437184) -> 36
# sum_digits(11408855402054064613470328848384) -> 126

def fact_iter(n):
    total, k = 1, 1
    while k <= n:
        total, k = total * k, k + 1
    return total            # fact_iter(4) -> 24

def fact(n):
    if n == 1:
        return 1
    else:
        return n * fact(n-1)   # fact(4) -> 24

def is_even(n):
    if n == 0:
        return True
    else:
        return is_odd(n-1)

def is_odd(n):
    if n == 0:
        return False
    else:
        return is_even(n-1)
# single-function version, abstraction boundary broken:
def is_even(n):
    if n == 0:
        return True
    else:
        if (n-1) == 0:
            return False
        else:
            return is_even((n-1)-1)

def cascade(n):
    """Print a cascade of prefixes of n."""
    if n < 10:
        print(n)
    else:
        print(n)
        cascade(n//10)
        print(n)
# compact form — base case need not come first:
def cascade(n):
    """Print a cascade of prefixes of n."""
    print(n)
    if n >= 10:
        cascade(n//10)
        print(n)

def play_alice(n):
    if n == 0:
        print("Bob wins!")
    else:
        play_bob(n-1)          # Alice always removes one pebble

def play_bob(n):
    if n == 0:
        print("Alice wins!")
    elif is_even(n):
        play_alice(n-2)        # Bob removes two if even on the table
    else:
        play_alice(n-1)
# play_alice(20) -> Bob wins!

def fib(n):
    if n == 1:
        return 0
    if n == 2:
        return 1
    else:
        return fib(n-2) + fib(n-1)   # fib(6) -> result = 5

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
# count_partitions(6, 4) -> 9;  (5,5) -> 7;  (10,10) -> 42
# (15,15) -> 176;  (20,20) -> 627
```

## Worked Examples

### 1. `sum_digits(738)` — the book's frame-by-frame walk
1. A local frame binds n=738; the body runs in the environment starting with that frame.
2. 738 is not < 10, so the assignment splits it: all_but_last=73, last=8.
3. The return statement calls `sum_digits` on 73 → a second local frame binds n=73.
4. 73 splits into 7 and 3; `sum_digits(7)` → a third frame binds n=7.
5. There `n < 10` holds, so 7 is returned (base case).
6. In the second frame, 7 + 3 → 10 returned; in the first, 10 + 8 → 18.
Each successive call takes a smaller argument until a single-digit input is reached — a complete computation with no new model rules.

### 2. `cascade(2013)` — printing on the way down and up
Output: 2013, 201, 20, 2, 20, 201, 2013. The base case (single digit) prints once; every other level prints its n, recurses on `n//10`, then prints n again — the first column of output happens descending, the mirror half ascending as frames return.

### 3. The pebble game — mutual recursion preserving abstraction
n pebbles; players alternate removing one or two; whoever removes the last pebble wins. Alice always removes one; Bob removes two if the count is even, else one. Each strategy lives in its own function (modifiable independently — the abstraction barrier); turn-taking is expressed by each function calling the other at the end of its turn. `play_alice(20)` → "Bob wins!". Note `play_bob` contains multiple recursive call *sites*, but each execution calls `play_alice` at most once — contrast with tree recursion.

### 4. `count_partitions(6, 4)` → 9 — tree recursion as possibility exploration
The 9 partitions: 2+4, 1+1+4, 3+3, 1+2+3, 1+1+1+3, 2+2+2, 1+1+2+2, 1+1+1+1+2, 1+1+1+1+1+1. Key observation: the number of ways to partition n using integers up to m equals (1) the ways to partition n−m using integers up to m, **plus** (2) the ways to partition n using integers up to m−1. Why: all partitions divide into those including at least one m (each is a partition of n−m with m appended — here the first two, which contain a 4) and those that do not. Two simpler problems: a smaller number, or smaller parts. Base cases: one way to partition 0 (no parts); 0 ways for negative n; 0 ways for n > 0 with parts of size 0. The first and second recursive calls correspond to "use a part of size m" and "don't"; a non-recursive implementation "would be substantially more involved."

## Common Errors (what the author explicitly flags)
- Omitting the special case: a one-digit number's digit sum is itself — without it `sum_digits` never terminates.
- Treating mutual recursion as magic — it reduces to simple recursion by inlining; its value is abstraction, not power.
- Using tree-recursive `fib` for performance: the earlier (iterative) definitions were more efficient.
- Forgetting any of the three partition base cases, especially n < 0, which the `n-m` call can produce.
- Expecting recursion to need new evaluation rules — the circularity worry is dissolved by def-time non-execution and per-call frames.

## Key Takeaways
1. Structure every recursive function as base case(s) + recursive call(s) that always simplify the problem.
2. Verify by the recursive leap of faith — trust the simpler case, check the combination; it's proof by induction.
3. Recursion stores computation state in the environment's frames (return values play `total`; frames play `k`); iteration stores it in explicit local names — recursion often avoids state-bookkeeping bugs.
4. Mutual recursion = simple recursion + preserved abstraction boundaries (one function per role/strategy).
5. Print before a recursive call to trace descent, after it to trace ascent.
6. Tree recursion explores possibilities (with-m / without-m); choose it when the branching decomposition makes the solution substantially simpler than iteration, and expect efficiency costs.
7. Enumerate all base cases explicitly, including degenerate ones (n == 0, n < 0, m == 0).

## Connects To
- **Section 1.3**: recursion works because `def` stores the body unevaluated and each call creates an independent local frame.
- **Section 1.5**: `fact_iter` and the iterative `fib` contrast iteration's explicit state with recursion's environmental state.
- **Section 1.2.5**: the evaluation procedure for call expressions was itself recursive; recursive functions apply the same idea at the program level.
- **Section 1.6**: the leap of faith is functional abstraction (domain/range/intent) applied to a function's own recursive call.
- **Chapter 2.8 (Efficiency)**: why tree-recursive `fib` is slow, and memoization.
