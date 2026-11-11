# Section 2.9: Recursive Objects

## Core Idea
Objects can have other objects as attribute values. When an object of some class has an attribute value of that *same* class, it is a **recursive object**. This section rebuilds the chapter's two workhorse recursive structures — linked lists and trees — as user-defined classes (`Link` and `Tree`), then uses them to implement sets three different ways. The payoffs are twofold: special method names (`__len__`, `__getitem__`, `__repr__`, `__add__`) let a recursive class interoperate with Python's built-in `len`, element selection, display, and `+`; and order-of-growth analysis (Θ) drives the choice among set representations — unordered list, ordered list, binary search tree — with Python's built-in hashed `set` as the end of the line.

## Frameworks Introduced
- **Link class**: `first` + `rest`, where `rest` is itself a `Link` or `Link.empty` (the empty tuple). Special methods make it a sequence: `len` and `[]` work via `__len__`/`__getitem__`, which are *indirectly recursive* — the built-in invokes the method, which invokes the built-in again on `self.rest`.
- **Closure property**: just as an element of a list can itself be a list, a `Link` can contain a `Link` as its `first` element.
- **repr as an evaluable expression**: `link_expression(s)` returns "a string that would evaluate to s"; assigning `Link.__repr__ = link_expression` makes the interpreter display links that way. Likewise `Link.__add__ = extend_link` emulates built-in list addition.
- **Recursive higher-order functions instead of comprehensions**: `map_link`, `filter_link`, and `join_link` express the same logic as list comprehensions and string `join` over linked lists.
- **Recursive construction**: linked lists shine when sequences are built incrementally inside recursive computations — `partitions(n, m)` enumerates integer partitions explicitly by the same analysis that `count_partitions` used to count them.
- **Tree class with internal values**: earlier trees held values only at the leaves; this `Tree` holds a **label** at the root of every subtree, with a sequence of `branches` that are also trees (asserted via `isinstance`).
- **Memoized construction / shared subtrees**: `fib_tree = memo(fib_tree)` creates each repeated subtree only once but uses it multiple times as a branch of different larger trees — identity (`is`) proves the sharing.
- **Sets as a data abstraction**: abstractly, a set is a collection of distinct objects supporting membership testing, union, intersection, and adjunction. Adjoining returns a *new* set containing the original elements plus the new one, if distinct. Any representation providing these behaviors is fair game; the section compares three by Θ analysis.

## Evaluation Rules
The author's formal machinery for this section:

1. **Recursive object definition**: an object of class C is recursive when one of its attribute values is an object of class C. The base case is a distinguished non-instance sentinel (`Link.empty = ()` for links; `None` for the empty BST).
2. **Link invariants**: `rest is Link.empty or isinstance(rest, Link)` is asserted in the constructor. `isinstance(rest, Link)` is true if `rest` is a `Link` instance *or an instance of some sub-class of* `Link`. The empty tuple is chosen as `Link.empty` because it already has length 0 and no elements.
3. **How built-ins dispatch**: `len(x)` on a user-defined object invokes `x.__len__()`; element selection `x[i]` (or `operator.getitem`) invokes `x.__getitem__(i)`. So `__len__`'s body `1 + len(self.rest)` calls itself indirectly through the built-in; the base case is reached when `self.rest` evaluates to `Link.empty`, which has length 0.
4. **Display rule**: Python displays instances of user-defined classes by invoking their `__repr__` method; the contract is a string that would evaluate back to an equal object. Addition `s + t` invokes `__add__`. Setting these as class attributes after the fact (`Link.__repr__ = link_expression`) works because methods are just class attributes.
5. **Tree structure**: a tree is any data structure that has as an attribute a sequence of branches that are also trees. Every branch is asserted `isinstance(branch, Tree)`; `is_leaf` means `not self.branches`. The internal value at the root of each subtree is called its label.
6. **BST invariant**: the `entry` of the root holds one element; *all* entries in the `left` branch are smaller than the root's `entry`, all entries in the `right` branch are larger. The same set may be represented by trees of many different shapes — the invariant, not the shape, defines validity.
7. **Θ accounting for the ordered-merge argument**: in each step of the ordered `intersect_set`, the size of at least one set shrinks; hence the number of steps is at most the *sum* of the sizes, not the product — Θ(n) rather than Θ(n²).
8. **BST Θ(log n) caveat**: the logarithmic claim *rests on the assumption that the tree is balanced* — each subtree holds about half the elements of its parent, so each comparison halves the problem.

## Worked Examples

### 1. Link: special methods, extension, map/filter/join, and partitions
Complete class and debugging display as the book gives them:

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

>>> s = Link(3, Link(4, Link(5)))
>>> len(s)
3
>>> s[1]
4
```

Trace `len(s)`: built-in `len` → `s.__len__()` → `1 + len(s.rest)` → … → `len(Link.empty)` = `len(())` = 0; total 3. Trace `s[1]`: `i != 0`, so return `s.rest[0]` → `s.rest.first` = 4.

Inspection and display:

```python
>>> def link_expression(s):
        """Return a string that would evaluate to s."""
        if s.rest is Link.empty:
            rest = ''
        else:
            rest = ', ' + link_expression(s.rest)
        return 'Link({0}{1})'.format(s.first, rest)
>>> link_expression(s)
'Link(3, Link(4, Link(5)))'
>>> Link.__repr__ = link_expression
>>> s
Link(3, Link(4, Link(5)))
```

Closure property in action — a two-element list whose first element is a three-element list:

```python
>>> s_first = Link(s, Link(6))
>>> s_first
Link(Link(3, Link(4, Link(5))), Link(6))
>>> len(s_first)
2
>>> len(s_first[0])
3
>>> s_first[0][2]
5
```

Addition via `extend_link`:

```python
>>> def extend_link(s, t):
        if s is Link.empty:
            return t
        else:
            return Link(s.first, extend_link(s.rest, t))
>>> Link.__add__ = extend_link
>>> s + s
Link(3, Link(4, Link(5, Link(3, Link(4, Link(5))))))
```

Higher-order replacements for comprehensions:

```python
>>> def map_link(f, s):
        if s is Link.empty:
            return s
        else:
            return Link(f(s.first), map_link(f, s.rest))
>>> def filter_link(f, s):
        if s is Link.empty:
            return s
        else:
            filtered = filter_link(f, s.rest)
            if f(s.first):
                return Link(s.first, filtered)
            else:
                return filtered
>>> odd = lambda x: x % 2 == 1
>>> map_link(square, filter_link(odd, s))
Link(9, Link(25))
>>> [square(x) for x in [3, 4, 5] if odd(x)]
[9, 25]

>>> def join_link(s, separator):
        if s is Link.empty:
            return ""
        elif s.rest is Link.empty:
            return str(s.first)
        else:
            return str(s.first) + separator + join_link(s.rest, separator)
>>> join_link(s, ", ")
'3, 4, 5'
```

Recursive construction — partitions of `n` using parts up to `m` either use `m` (partition `n-m` up to `m`, then prepend `m` to each) or don't (partition `n` up to `m-1`). Base cases: 0 has one empty partition; a negative `n` or `m == 0` has none:

```python
>>> def partitions(n, m):
        """Return a linked list of partitions of n using parts of up to m.
        Each partition is represented as a linked list."""
        if n == 0:
            return Link(Link.empty)  # A list containing the empty partition
        elif n < 0 or m == 0:
            return Link.empty
        else:
            using_m = partitions(n-m, m)
            with_m = map_link(lambda s: Link(m, s), using_m)
            without_m = partitions(n, m-1)
            return with_m + without_m

>>> def print_partitions(n, m):
        lists = partitions(n, m)
        strings = map_link(lambda s: join_link(s, " + "), lists)
        print(join_link(strings, "\n"))
>>> print_partitions(6, 4)
4 + 2
4 + 1 + 1
3 + 3
3 + 2 + 1
3 + 1 + 1 + 1
2 + 2 + 2
2 + 2 + 1 + 1
2 + 1 + 1 + 1 + 1
1 + 1 + 1 + 1 + 1 + 1
```

### 2. Tree: internal labels, fib_tree, sum_labels, memoized sharing

```python
>>> class Tree:
        def __init__(self, label, branches=()):
            self.label = label
            for branch in branches:
                assert isinstance(branch, Tree)
            self.branches = branches
        def __repr__(self):
            if self.branches:
                return 'Tree({0}, {1})'.format(self.label, repr(self.branches))
            else:
                return 'Tree({0})'.format(repr(self.label))
        def is_leaf(self):
            return not self.branches

>>> def fib_tree(n):
        if n == 1:
            return Tree(0)
        elif n == 2:
            return Tree(1)
        else:
            left = fib_tree(n-2)
            right = fib_tree(n-1)
            return Tree(left.label + right.label, (left, right))
>>> fib_tree(5)
Tree(3, (Tree(1, (Tree(0), Tree(1))), Tree(2, (Tree(1), Tree(1, (Tree(0), Tree(1)))))))
```

`fib_tree(n)` has the nth Fibonacci number as its label and a trace of all previously computed Fibonacci numbers within its branches. Processing is again recursive — summing labels, with an empty branches tuple as the base case:

```python
>>> def sum_labels(t):
        """Sum the labels of a Tree instance, which may be None."""
        return t.label + sum([sum_labels(b) for b in t.branches])
>>> sum_labels(fib_tree(5))
10
```

Memoizing the *constructor function* turns the computation tree into a DAG of shared objects:

```python
>>> fib_tree = memo(fib_tree)
>>> big_fib_tree = fib_tree(35)
>>> big_fib_tree.label
5702887
>>> big_fib_tree.branches[0] is big_fib_tree.branches[1].branches[1]
True
>>> sum_labels = memo(sum_labels)
>>> sum_labels(big_fib_tree)
142587180
```

The savings are substantial: instead of creating 18,454,929 different `Tree` instances, we create only 35. Memoizing `sum_labels` too means each shared subtree is summed once.

### 3. Sets three ways: unordered list, ordered list, binary search tree

**Unordered sequences** — a sequence in which no element appears more than once; the empty set is the empty sequence:

```python
>>> def empty(s):
        return s is Link.empty

>>> def set_contains(s, v):
        """Return True if and only if set s contains v."""
        if empty(s):
            return False
        elif s.first == v:
            return True
        else:
            return set_contains(s.rest, v)
>>> s = Link(4, Link(1, Link(5)))
>>> set_contains(s, 2)
False

>>> def adjoin_set(s, v):
        """Return a set containing all elements of s and element v."""
        if set_contains(s, v):
            return s
        else:
            return Link(v, s)
>>> t = adjoin_set(s, 2)
>>> t
Link(2, Link(4, Link(1, Link(5))))

>>> def intersect_set(set1, set2):
        """Return a set containing all elements common to set1 and set2."""
        return keep_if_link(set1, lambda v: set_contains(set2, v))

>>> def union_set(set1, set2):
        """Return a set containing all elements either in set1 or set2."""
        set1_not_set2 = keep_if_link(set1, lambda v: not set_contains(set2, v))
        return extend_link(set1_not_set2, set2)
```

Membership is Θ(n) on average; adjoin, built on it, is also linear. Intersection tests each element of `set1` for membership in `set2` — a quadratic Θ(n²) process for two sets of size n — and union likewise requires a linear number of membership tests, also Θ(n²).

**Ordered sequences** — list elements in increasing order (concentrating on numbers, compared with `<` and `>`). Membership can stop early:

```python
>>> def set_contains(s, v):
        if empty(s) or s.first > v:
            return False
        elif s.first == v:
            return True
        else:
            return set_contains(s.rest, v)
```

Worst case is unchanged (the sought item may be the largest), but on average we examine about half the items — about n/2 steps. Still Θ(n), but a real practical saving. The impressive speedup is intersection by simultaneous iteration: when `e1 == e2`, include it; when `e1 < e2`, `e1` cannot appear in the remainder of `set2`, so discard it (and symmetrically for `e2 < e1`):

```python
>>> def intersect_set(set1, set2):
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

Each step shrinks at least one set, so steps ≤ size(set1) + size(set2): Θ(n), not Θ(n²). Two sets of size 100 intersect in about 200 steps rather than 10,000. Adjunction and union for ordered sequences can also be computed in linear time (left as an exercise).

**Binary search trees** — two-branch trees where the root's `entry` holds one element, the `left` branch holds all smaller elements, the `right` all larger. Note the empty BST is `None` here, and the `Tree` in this subsection carries `entry`/`left`/`right`:

```python
>>> def set_contains(s, v):
        if s is None:
            return False
        elif s.entry == v:
            return True
        elif s.entry < v:
            return set_contains(s.right, v)
        elif s.entry > v:
            return set_contains(s.left, v)

>>> def adjoin_set(s, v):
        if s is None:
            return Tree(v)
        elif s.entry == v:
            return s
        elif s.entry < v:
            return Tree(s.entry, s.left, adjoin_set(s.right, v))
        elif s.entry > v:
            return Tree(s.entry, adjoin_set(s.left, v), s.right)
>>> adjoin_set(adjoin_set(adjoin_set(None, 2), 3), 1)
Tree(2, Tree(1), Tree(3))
```

Each comparison restricts the search to one subtree; if the tree is balanced, each subtree is about half the size, so both contains and adjoin take Θ(log n) steps. Adjoin is functional: it rebuilds only the path from the root to the insertion point, piecing the newly constructed branch together with the original `entry` and the other branch. Intersection and union on tree-structured sets can be performed in linear time by converting them to ordered lists and back (details left as an exercise).

**Python's built-in set** uses none of these representations. It uses *hashing*, giving constant-time membership tests and adjoin operations (a topic for another course). Literals follow mathematical notation — `{3, 2, 1, 4, 4}` displays as `{1, 2, 3, 4}`: duplicates removed, unordered. Beyond `union`/`intersection`: predicates `isdisjoint`, `issubset`, `issuperset`; single-element mutators `add`, `remove`, `discard`, `pop`; multi-element mutations `clear` and `update`. Built-in sets cannot contain mutable data types (lists, dictionaries, or other sets); `frozenset` shares methods with `set` but excludes mutation methods and operators, allowing nested sets.

## Key Concepts
- **Recursive object**: an instance whose attribute value is an instance of the same class.
- **Link.empty**: the empty tuple sentinel for linked lists; always tested with `is`.
- **Indirect recursion through built-ins**: `__len__` calls `len(self.rest)`, which re-invokes `__len__` — the recursion threads through Python's dispatch.
- **isinstance**: true when the first argument's type is or inherits from the second — the constructor assertion accepts subclasses of `Link`.
- **Closure property**: a `Link` can contain a `Link` as its `first` element.
- **repr contract**: `__repr__` returns a string expression that would evaluate to the object.
- **Label**: the internal value at the root of each subtree in the `Tree` class.
- **Shared subtrees**: memoized constructors return the *same object* for repeated arguments; `is` detects it.
- **Adjunction**: produces a new set with the element added if distinct — all three implementations are non-mutating.
- **BST entry/left/right**: the ordering invariant that enables binary search; many tree shapes represent the same set.
- **Balance**: the condition (subtrees of roughly equal size) under which BST operations are Θ(log n).
- **Hashing / frozenset**: Python's constant-time set machinery and its immutability requirement for elements.

## Mental Models
- Special methods are the adapter layer: implement `__len__`, `__getitem__`, `__repr__`, `__add__` and your recursive class behaves like a built-in sequence to all existing code.
- Write `__repr__` by asking "what expression would rebuild this?" — the display is executable documentation.
- Memoization on a constructor turns a tree of computations into a DAG of shared objects; equality of results becomes identity of objects.
- A set representation is a contract (membership, adjoin, union, intersection) plus a Θ profile. Ordering buys fast intersection (merge), tree structure buys fast lookup (halving), hashing buys both — each at the price of a stronger invariant to maintain.
- The ordered-merge argument generalizes: when each step consumes an element from at least one input, total work is bounded by the sum of input sizes.

## Anti-patterns
- **Scanning unordered lists for intersection/union**: Θ(n²) — the complete scan of `set2` per element of `set1` is exactly what ordering eliminates (10,000 steps vs ~200 at n=100).
- **Trusting BST balance**: random insertion balances only *on average*; it is not a guarantee.
- **Putting mutable values in built-in sets**: lists, dictionaries, and sets cannot be elements; nest with `frozenset`.
- **Conflating the two Trees**: the labeled general `Tree` (label + branches tuple) and the BST (`entry`/`left`/`right`, empty = `None`) are different structures in this section; mixing their protocols breaks both.

## Common Errors
Mistakes the author explicitly flags or guards against:
- **Skipping the constructor assertion**: without `assert rest is Link.empty or isinstance(rest, Link)`, malformed lists fail far from where they were built. The author validates the recursive structure at construction.
- **Double-counting in union**: "we must be careful not to include any element twice" — hence `union_set` first filters `set1` down to elements *not* in `set2` before extending.
- **Assuming ordered search is asymptotically faster**: early stopping saves ~half the steps on average, but worst case equals the unordered case; the growth is still Θ(n). Only the *intersection* algorithm changes the Θ class.
- **Assuming BSTs stay balanced**: "this is not a guarantee" — adjoining 1 through 7 in sequence yields a highly unbalanced tree with all left subtrees empty, "no advantage over a simple ordered list." The fix the author names: define an operation that transforms an arbitrary tree into a balanced tree with the same elements, and apply it after every few `adjoin_set` operations.
- **Expecting an uninspectable class to debug itself**: a raw `Link` instance "is currently difficult to inspect" — define the string conversion (`link_expression` → `__repr__`) early, for debugging.

## Reference Tables
Θ comparison across the three implementations (two sets of size n for the binary operations), as the source gives them:

| Representation | membership (`set_contains`) | adjoin | intersection | union |
|---|---|---|---|---|
| Unordered sequence (Link) | Θ(n) | Θ(n) | Θ(n²) | Θ(n²) |
| Ordered sequence (Link) | Θ(n), ~n/2 steps on average | linear (exercise) | Θ(n) by simultaneous iteration | linear (exercise) |
| Binary search tree | Θ(log n) *if balanced* | Θ(log n) *if balanced* | linear, via ordered lists and back (exercise) | linear, via ordered lists and back (exercise) |
| Python built-in `set` (hashing) | constant time | constant time | — | — |

## Key Takeaways
1. Model recursive data as classes whose attributes are instances of the same class, with the base case as a sentinel (`Link.empty = ()`; `None` for the empty BST) and the invariant asserted in the constructor.
2. Implement `__len__`, `__getitem__`, `__repr__`, `__add__` so recursive classes interoperate with built-ins; `__repr__` returns an expression that would evaluate to the object; recursion runs indirectly through the built-in dispatch.
3. Recursive higher-order functions (`map_link`, `filter_link`, `join_link`) replace comprehensions and `join` for linked structures, and compose into real algorithms (`partitions`).
4. Memoize recursive constructors (and the functions that traverse their results) to share identical substructures — 35 `Tree` instances instead of 18,454,929 for `fib_tree(35)`.
5. A set is a data abstraction; representation choice is a Θ decision: ordering turns quadratic intersection linear, balanced BSTs give logarithmic lookup and adjoin, and hashing gives constant time at the cost of element immutability.
6. Maintain the invariant that makes the representation fast — sortedness or balance — or the advantage silently vanishes; rebalance BSTs periodically.

## Connects To
- **Section 2.3**: functional linked lists and trees, now as classes; `keep_if_link`/`apply_to_all_link` from there appear in the unordered-set code; `partitions` mirrors Chapter 1's `count_partitions`.
- **Section 2.7**: special methods (`__repr__`, `__len__`, `__getitem__`, `__add__`) applied; the repr contract made concrete.
- **Section 2.8**: `memo` and orders of growth drive the shared-subtree trick and every representation comparison here.
- **Section 2.5**: class attributes (`Link.empty`) and assigning functions as methods after class definition.
