# Section 2.3: Sequences

## Core Idea
A sequence is not a particular type but a collection of behaviors shared by many types — the **sequence abstraction** — defined minimally by length and element selection. Whole programs can be organized as pipelines of simple, focused sequence-processing operations.

## Frameworks Introduced
- **Sequence abstraction**: the shared interface over lists, ranges, strings, and user-defined types.
  - When to use: whenever you design a new ordered collection or write code generic over collections.
  - How: at minimum provide (1) **Length** — finite, empty = 0; (2) **Element selection** — an element for every index 0 ≤ i < length. Python's full abstraction adds **Membership** (`in`/`not in`) and **Slicing** (`s[a:b]`, end-exclusive, omitted bounds default to extremes).
- **Sequence processing patterns** (map / filter / aggregate): most sequence computations combine three patterns.
  - When to use: structuring data-processing code.
  - How: evaluate an expression per element (list comprehension / `apply_to_all` a.k.a. **map**); select a subset (`keep_if` a.k.a. **filter**); aggregate to one value (`sum`, `min`, `max`, `reduce(reduce_fn, s, initial)`).
- **List comprehension**: `[<map expression> for <name> in <sequence expression> if <filter expression>]` — evaluate the sequence expression, bind each element, keep map-expression values where the filter is true.
- **Closure property**: a means of combination has the closure property if results of combination can themselves be combined by the same method (lists inside lists). Closure is the key to power in any means of combination — it permits hierarchical structures.
- **Tree data abstraction**: constructor `tree(root_label, branches)` and selectors `label(t)`, `branches(t)`; every branch is itself a tree; a tree with no branches is a **leaf**.
- **Linked list representation**: a sequence as nested pairs — `link(first, rest)` with selectors `first(s)`, `rest(s)` and the sentinel `empty`; recursive structure (the rest of a linked list is a linked list or `'empty'`).
- **Rich vs simple abstractions**: richer abstractions help users but are harder to implement for new types and longer to learn; sequences earn their richness by ubiquity — most user-defined abstractions should be kept as simple as possible.

## Key Concepts
- **for statement**: `for <name> in <expression>:` — evaluates the header to an **iterable value**, binds name to each element in order (in the current frame), executes the suite; the name remains bound to the last element afterward.
- **Sequence unpacking**: multiple names in a for header (`for x, y in pairs`) unpack fixed-length element sequences.
- **range**: a native sequence of integers, start inclusive, end exclusive; `for _ in range(3)` with the conventional underscore for an unused name.
- **Slicing**: `digits[0:2]`, `digits[1:]` — a contiguous span; a new sequence.
- **String**: native text type; elements are length-1 strings (no separate char type); supports `+`, `*`, `len`, indexing; `in` on strings matches *substrings*, diverging from the sequence abstraction; `str(obj)` coerces any object.
- **Box-and-pointer notation**: lists drawn as adjacent boxes; composite values shown as arrows.
- **Node / sub-tree**: any tree contained in a tree is a sub-tree; the root of each sub-tree is a node.
- **Binarization**: transforming a tree into a binary tree by grouping adjacent branches (`right_binarize`).
- **map / filter / reduce**: conventional names; built-ins `map`/`filter` return non-lists, `functools.reduce` has optional initial.

## Mental Models
- Think of sequence processing as a pipeline: modular components with sequences as both inputs and outputs can be mixed and matched.
- Use the three-pattern decomposition when facing a sequence problem: map each element? filter a subset? aggregate? Perfect numbers = filter(range) by aggregate(map).
- Think of a linked list computation as operating on an increasingly shorter suffix of the list — both iteratively (`s = rest(s)`) and recursively (recurse on `rest(s)`).
- Think of trees recursively: a function on a tree handles the leaf base case and combines recursive results over `branches(t)` (e.g. `count_leaves` sums branch counts).

## Anti-patterns
- **Assuming strings follow the full sequence abstraction**: `in` on strings matches substrings, not elements — `'here' in "Where's Waldo?"` is True.
- **Over-enriching user-defined abstractions**: rich interfaces are harder to satisfy and learn; keep them minimal.
- **Expecting cheap length/indexing from linked lists**: each access walks the chain; Python's built-in sequences are implemented differently to avoid that cost.
- **Float-style off-by-one with range/slices**: both are end-exclusive; `range(1, 10)` includes 1, not 10.

## Code Examples
```python
def divisors(n):
    return [1] + [x for x in range(2, n) if n % x == 0]

>>> [n for n in range(1, 1000) if sum(divisors(n)) == n]
[6, 28, 496]

def partitions(n, m):
    """Return a linked list of partitions of n using parts of up to m."""
    if n == 0:
        return link(empty, empty)  # A list containing the empty partition
    elif n < 0 or m == 0:
        return empty
    else:
        using_m = partitions(n-m, m)
        with_m = apply_to_all_link(lambda s: link(m, s), using_m)
        without_m = partitions(n, m-1)
        return extend_link(with_m, without_m)
```
- **What it demonstrates**: map/filter/aggregate solving perfect numbers in two lines, and recursive construction of sequences of sequences with linked lists.

## Worked Example
Linked lists end to end. Represent `[1, 2, 3, 4]` as `[1, [2, [3, [4, 'empty']]]]`. Define the abstraction: `is_link` validates, `link(first, rest)` constructs (asserting rest is a link), `first`/`rest` select; behavior condition: constructor and selectors are inverse functions. Prove it satisfies the sequence abstraction by implementing `len_link` and `getitem_link` (iterative: `s, i = rest(s), i-1`) and their recursive twins. Build recursive transformers: `extend_link` (concatenate), `apply_to_all_link` (map), `keep_if_link` (filter), `join_link` (string aggregation). Culminates in `partitions(n, m)`: either partition `n-m` with parts up to `m` (prepend `m` to each result) or partition `n` with parts up to `m-1`; `print_partitions(6, 4)` prints all nine partitions via `join_link`. The companion `partition_tree`/`print_parts` solves the same problem as a binary tree with True/False leaves marking successful paths.

## Key Takeaways
1. Program against the sequence abstraction (length + element selection), not against a specific type.
2. Decompose sequence problems into map, filter, and aggregate; list comprehensions are the idiomatic Python form, higher-order functions the equivalent general form.
3. Closure (lists in lists) is what makes hierarchical data possible; trees impose regularity on that hierarchy via `tree`/`label`/`branches`/`is_leaf`.
4. Tree problems are solved by tree-recursive functions: base case at leaves, recursive combination over branches.
5. Linked lists show that a full sequence type can be built from pairs alone (hence from functions alone), and they shine for incremental recursive construction.
6. Keep user-defined abstractions simple; richness must be earned by ubiquity.

## Connects To
- **Section 2.2**: trees and linked lists are data abstractions built with constructors/selectors and behavior conditions.
- **Section 2.9**: linked lists and trees return as classes (`Link`, `Tree`) once OOP is available.
- **Section 2.8**: the cost of linked-list length/indexing previews orders of growth.
