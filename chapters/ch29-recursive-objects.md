# Section 2.9: Recursive Objects

## Core Idea
A **recursive object** is an object of a class whose attribute values are objects of that same class — linked lists (`Link`) and trees (`Tree`) as user-defined classes. Recursive class structure plus special methods lets these types plug into Python's built-in `len`, indexing, and `+`, and supports Θ-driven choices among set representations.

## Frameworks Introduced
- **Link class**: `first` + `rest` (itself a Link or `Link.empty = ()`, the empty tuple); `__len__` returns `1 + len(self.rest)` (base case: empty tuple has length 0); `__getitem__(i)` returns `first` at 0 else recurses on `rest[i-1]` — built-ins become indirectly recursive.
  - Closure property: a `Link` can contain a `Link` as its `first` element.
- **repr via assignment**: write `link_expression(s)` returning a string that would evaluate to s (`'Link(3, Link(4, Link(5)))'`), then `Link.__repr__ = link_expression`; likewise `Link.__add__ = extend_link` emulates built-in list addition.
- **map_link / filter_link / join_link**: recursive higher-order replacements for list comprehensions on linked lists; `map_link(square, filter_link(odd, s))` ≡ `[square(x) for x in [3,4,5] if odd(x)]`.
- **Tree class with internal values**: each tree has a `label` and a tuple of `branches` that are also trees (asserted); `is_leaf` = no branches; earlier trees held values only at leaves, this one holds a label at every node.
- **Memoized tree construction / shared subtrees**: `fib_tree = memo(fib_tree)` makes repeated subtrees *the same object* reused in multiple parents (`branches[0] is branches[1].branches[1]` → True); 35 Tree instances instead of 18,454,929 for fib_tree(35).
- **Three set representations (sets as a data abstraction**: distinct objects supporting membership, union, intersection, adjunction**)**:
  1. **Unordered sequences** (Link): contains Θ(n), adjoin Θ(n), intersection/union Θ(n²).
  2. **Ordered sequences**: contains still Θ(n) but ~n/2 on average (stop when `first > v`); intersection by simultaneous iteration — each step shrinks a set, so at most size(set1)+size(set2) steps = Θ(n); adjoin/union also linear.
  3. **Binary search trees**: left branch holds elements smaller than the root `entry`, right holds larger; contains and adjoin halve the problem each step → Θ(log n) *if balanced*; the same set has many valid tree shapes.

## Key Concepts
- **Link.empty**: the empty tuple sentinel; test with `is`.
- **isinstance in the constructor**: `assert rest is Link.empty or isinstance(rest, Link)` validates the recursive structure at construction.
- **Adjunction**: adjoining v returns a new set with v added if distinct (functional, non-mutating).
- **BST adjoin**: rebuild the path — `Tree(s.entry, s.left, adjoin_set(s.right, v))` when entry < v.
- **Balance**: the Θ(log n) claim assumes balanced trees; random insertion tends to balance on average, but inserting 1..7 in order yields a degenerate tree no better than an ordered list; fix by rebalancing after every few adjoins.
- **BST intersection/union**: linear time by converting to ordered lists and back.
- **Python's built-in set**: `{3, 2, 1, 4, 4}` dedupes, unordered; union/intersection, `isdisjoint`/`issubset`/`issuperset`, mutators `add`/`remove`/`discard`/`pop`/`clear`/`update`; implemented by **hashing** — constant-time membership and adjoin; elements must be immutable (`frozenset` for nested sets).

## Mental Models
- Think of `__len__`/`__getitem__`/`__add__`/`__repr__` as the adapter layer: implement them and your recursive class behaves like a built-in sequence.
- Write `__repr__` as "a string that would evaluate to this object" — the repr contract made concrete.
- Memoization on constructors turns a tree of computations into a DAG of shared objects: identity (`is`) proves the sharing.
- Choose set representation by the operation mix and Θ table below; ordering buys intersection speed, tree structure buys lookup speed, hashing buys both (at the cost of immutability requirements).

## Anti-patterns
- **Scanning unordered lists for intersection**: Θ(n²) — two sets of size 100 cost ~10,000 steps vs ~200 with ordered merge.
- **Trusting BST balance**: sequential insertion degenerates to a linked list; rebalance periodically.
- **Putting mutable values in Python sets**: lists, dicts, and sets cannot be elements; use `frozenset` to nest.

## Code Examples
```python
class Link:
    """A linked list with a first element and the rest."""
    empty = ()
    def __init__(self, first, rest=empty):
        assert rest is Link.empty or isinstance(rest, Link)
        self.first = first
        self.rest = rest
    def __getitem__(self, i):
        if i == 0:
            return self.first
        else:
            return self.rest[i-1]
    def __len__(self):
        return 1 + len(self.rest)

def intersect_set(set1, set2):          # ordered representation, Θ(n)
    if empty(set1) or empty(set2):
        return Link.empty
    else:
        e1, e2 = set1.first, set2.first
        if e1 == e2:
            return Link(e1, intersect_set(set1.rest, set2.rest))
        elif e1 < e2:
            return intersect_set(set1.rest, set2)
        elif e2 < e1:
            return intersect_set(set1, set2.rest)
```
- **What it demonstrates**: a recursive class joining built-in protocols, and the ordered-merge intersection that drops Θ(n²) to Θ(n).

## Reference Tables
| Representation | contains | adjoin | intersect / union |
|---|---|---|---|
| Unordered sequence | Θ(n) | Θ(n) | Θ(n²) |
| Ordered sequence | Θ(n) (~n/2 avg) | Θ(n) | Θ(n) |
| Binary search tree (balanced) | Θ(log n) | Θ(log n) | Θ(n) via ordered lists |
| Python built-in set (hashing) | ~Θ(1) | ~Θ(1) | — |

## Worked Example
The Link class end to end. Build `s = Link(3, Link(4, Link(5)))`; `len(s)` → 3 and `s[1]` → 4 through the recursive special methods. Attach `link_expression` as `__repr__` so the interpreter displays `Link(3, Link(4, Link(5)))`, and `extend_link` as `__add__` so `s + s` concatenates. Reimplement map/filter/join recursively, then reuse them for `partitions(n, m)` exactly as in 2.3 — base cases `Link(Link.empty)` (one empty partition) and `Link.empty` (impossible); recursive case prepends `m` to each partition of `n-m` via `map_link(lambda s: Link(m, s), using_m)` and concatenates with partitions using parts up to `m-1`; `print_partitions(6, 4)` prints all nine partitions. The Tree class parallels this: `fib_tree(n)` labels each node with a Fibonacci number; `sum_labels` recursively sums; memoizing both makes fib_tree(35) build 35 nodes with shared subtrees.

## Key Takeaways
1. Model recursive data as classes whose attributes are instances of the same class, with the base case as a class-level sentinel (`Link.empty`).
2. Implement `__len__`, `__getitem__`, `__repr__`, `__add__` to make recursive classes interoperate with built-ins; `__repr__` should return an evaluable expression.
3. Recursive higher-order functions (map/filter/join) replace comprehensions for linked structures.
4. Memoize recursive constructors to share identical substructures — massive time and memory savings.
5. Representation choice is a Θ decision: ordering turns quadratic intersection linear; balanced BSTs give logarithmic lookup; hashing (built-in set) gives constant time.
6. Maintain the invariant that makes the representation fast (sortedness, balance) or the advantage vanishes.

## Connects To
- **Section 2.3**: the functional linked lists and trees, now as classes; partitions reappears verbatim.
- **Section 2.7**: special methods (`__repr__`, `__len__`, `__getitem__`, `__add__`) in action.
- **Section 2.8**: memoization and orders of growth drive every representation comparison here.
