# Section 2.3: Sequences

## Core Idea
A sequence is an ordered collection of values — but it is not a particular built-in type or abstract data representation. It is a **collection of behaviors shared among several different types of data**: the sequence abstraction. Lists, ranges, strings, and user-defined representations (linked lists) all qualify by exhibiting the same minimal behaviors. Because sequences are so common a form of compound data, whole programs are often organized around this single abstraction: modular components that take sequences in and produce sequences out can be mixed and matched into pipelines of simple, focused operations.

## Frameworks Introduced
- **Sequence abstraction**: the shared interface over lists, ranges, strings, and user-defined types.
  - When to use: whenever you design a new ordered collection or write code generic over collections.
  - How: at minimum provide (1) **Length** and (2) **Element selection** (see Evaluation Rules). Python's full abstraction adds **Membership** (`in`/`not in`) and **Slicing**.
- **Sequence processing patterns** (map / filter / aggregate): most sequence computations combine three patterns.
  - When to use: structuring any data-processing code.
  - How: evaluate an expression per element (`apply_to_all`, conventionally **map**); select a subset (`keep_if`, conventionally **filter**); aggregate to one value (`sum`, `min`, `max`, or the general `reduce(reduce_fn, s, initial)`).
- **List comprehension**: the idiomatic Python expression combining map and filter in one form (see Evaluation Rules).
- **Closure property**: a method for combining data values has a closure property if the result of combination can itself be combined using the same method (lists containing lists). Closure is the key to power in any means of combination because it permits hierarchical structures — structures made up of parts, which themselves are made up of parts, and so on.
- **Tree data abstraction**: constructor `tree(root_label, branches)` and selectors `label(t)`, `branches(t)`; every branch is itself a tree; a tree with no branches is a **leaf**. The tree imposes regularity on how hierarchical values are structured and manipulated.
- **Linked list representation**: a sequence as nested pairs — `link(first, rest)` with selectors `first(s)`, `rest(s)` and the sentinel `empty = 'empty'`. Recursive structure: the rest of a linked list is a linked list or `'empty'`. Demonstrates that a full sequence type can be built outside the language's native types.
- **Rich vs simple abstractions**: additional behaviors help users, but satisfying the requirements of a rich abstraction with a new data type is challenging, and rich abstractions take longer to learn. Sequences earn their richness by ubiquity; in general, most user-defined abstractions should be kept as simple as possible.

## Evaluation Rules

**Sequence abstraction — defining behaviors.** All sequences share:
1. **Length.** A sequence has a finite length. An empty sequence has length 0.
2. **Element selection.** A sequence has an element corresponding to any non-negative integer index less than its length, starting at 0 for the first element.

Python extends the abstraction for its native sequence types:
3. **Membership.** The operators `in` and `not in` evaluate to `True` or `False` depending on whether an element appears in the sequence. (Strings deviate: see Common Errors.)
4. **Slicing.** A slice is any contiguous span of the original sequence, designated by a pair of integers in square brackets separated by a colon. As with `range`, the first integer is the starting index and the second is one beyond the ending index. Any omitted bound is assumed to be an extreme value: 0 for the start, the length of the sequence for the end. `digits[0:2]` → `[1, 8]`; `digits[1:]` → `[8, 2, 8]`.

**For statement.** Form:
```
for <name> in <expression>:
    <suite>
```
Execution procedure:
1. Evaluate the header `<expression>`, which must yield an **iterable value**.
2. For each element value in that iterable value, in order:
   a. Bind `<name>` to that value **in the current frame** (no new frame).
   b. Execute the `<suite>`.

Consequence: `<name>` remains bound to the **last** element after the `for` statement finishes — the `for` loop is another way a statement updates the environment. Lists are sequences, sequences are iterable values, and their elements are considered in sequential order (the general definition of "iterable" waits for Chapter 4's iterators).

**Sequence unpacking.** When elements are themselves fixed-length sequences, a `for` header may include multiple names to unpack each element into its parts: `for x, y in pairs:` binds `x` and `y` to the first and second elements of each pair. This is the same pattern as multiple assignment (`a, b = 1, 2`).

**Ranges.** `range(a, b)` represents the integers from `a` (inclusive) to `b` (exclusive). With one argument, the range starts at 0: `list(range(4))` → `[0, 1, 2, 3]`. Ranges commonly appear in a `for` header to run a suite a fixed number of times; the conventional name for an unused loop variable is a single underscore `_` — just another name to the interpreter, but a signal among programmers that the name will not appear in later expressions.

**List comprehension.** General form:
```
[<map expression> for <name> in <sequence expression> if <filter expression>]
```
Evaluation procedure: evaluate the `<sequence expression>`, which must return an iterable value. Then, for each element in order: bind the element to `<name>`, evaluate the filter expression, and if it yields a true value, evaluate the map expression. The values of the map expressions are collected into a list. The `for` keyword here is part of the comprehension (it is inside square brackets), not a `for` statement.

**Linked list behavior condition.** Like a pair, the constructor and selectors are inverse functions: if a linked list `s` was constructed from first element `f` and linked list `r`, then `first(s)` returns `f` and `rest(s)` returns `r`.

**Partition tree structure.** A partition tree for `n` using parts up to size `m` is a binary tree representing computational choices. In a non-leaf partition tree: the left (index 0) branch contains all ways of partitioning `n` using at least one `m`; the right (index 1) branch contains partitions using parts up to `m-1`; the root label is `m`. Leaf labels (`True`/`False`) express whether the root-to-leaf path represents a successful partition of `n`.

## Sequence Processing Patterns (reference table)

| Pattern | Comprehension form | Higher-order form | Conventional name | Python built-in |
|---|---|---|---|---|
| Per-element expression | `[f(x) for x in s]` | `apply_to_all(map_fn, s)` | **map** | `list(map(f, s))` |
| Select a subset | `[x for x in s if p(x)]` | `keep_if(filter_fn, s)` | **filter** | `list(filter(p, s))` |
| Aggregate to one value | — | `reduce(reduce_fn, s, initial)` | **reduce** | `functools.reduce` (initial optional); `sum`, `min`, `max` |

The built-in `map` and `filter` are generalizations that do not return lists (Chapter 4); the definitions here equal applying `list` to their results. In Python programs it is more common to use list comprehensions directly, but both approaches are widely used.

```python
>>> def apply_to_all(map_fn, s):
        return [map_fn(x) for x in s]
>>> def keep_if(filter_fn, s):
        return [x for x in s if filter_fn(x)]
>>> def reduce(reduce_fn, s, initial):
        reduced = initial
        for x in s:
            reduced = reduce_fn(reduced, x)
        return reduced
>>> reduce(mul, [2, 4, 8], 1)
64
>>> apply_to_all = lambda map_fn, s: list(map(map_fn, s))
>>> keep_if = lambda filter_fn, s: list(filter(filter_fn, s))
>>> from functools import reduce
>>> from operator import mul
>>> def product(s):
        return reduce(mul, s)
>>> product([1, 2, 3, 4, 5])
120
```

## Key Concepts
- **list**: a sequence of arbitrary length with a large set of built-in behaviors; `+` concatenates, `*` with an integer replicates (`[2, 7] + digits * 2`); any values can be elements, including other lists (`pairs[1][0]`).
- **for statement**: see Evaluation Rules; binds in the current frame; name stays bound afterward.
- **Sequence unpacking**: multiple names in a for header to unpack fixed-length element sequences.
- **range**: native integer sequence, start inclusive, end exclusive; `for _ in range(3)` runs a suite three times.
- **Slicing**: contiguous span, end-exclusive, omitted bounds default to extremes; also works on a tree's branches (`tree[1:]`).
- **String**: native text type (`str`); elements are length-1 strings — Python has no separate character type; supports `+`, `*`, `len`, indexing; triple quotes delimit multiline literals where `\n` is a single element; `str(obj)` coerces any object to a descriptive string.
- **Box-and-pointer notation**: lists drawn as adjacent boxes; primitive values (numbers, strings, booleans, `None`) sit inside element boxes; composite values (functions, other lists) are indicated by arrows.
- **Closure property**: results of combination can themselves be combined by the same method.
- **Node / sub-tree / leaf**: any tree contained within a tree is a sub-tree; the root of each sub-tree is a node of the enclosing tree; a tree with no branches is a leaf.
- **Binarization**: transforming a tree into a binary tree (at most two branches) by grouping adjacent branches (`right_binarize`).
- **map / filter / reduce**: conventional names for `apply_to_all` / `keep_if` / aggregation.
- **Linked list**: nested (first, rest) pairs ending in `'empty'`; satisfies the sequence abstraction via `len_link` / `getitem_link`.

## Mental Models
- Think of sequence processing as a pipeline: complex components are defined by chaining a pipeline of sequence-processing operations, each simple and focused.
- Facing a sequence problem, run the three-pattern decomposition: map each element? filter a subset? aggregate? Perfect numbers = filter a range by an aggregate of a map.
- Think of a linked-list computation as operating on an increasingly shorter suffix of the original list — iteratively (`s = rest(s)` each step) or recursively (recurse on `rest(s)`). Each access peels one layer of nested pair.
- Think of trees recursively: a tree function handles the leaf base case and combines recursive results over `branches(t)` (e.g. `count_leaves` returns 1 at a leaf, else the sum of branch counts).
- Partition problems have one recursive shape in two clothings: `partitions` (linked lists of results) and `partition_tree`/`print_parts` (binary tree of choices) both split on "use at least one `m`" vs "use parts up to `m-1`".

## Worked Examples

### 1. Counting occurrences: while → for → the general patterns
Count how many times a value appears in a sequence. First with a `while` loop and explicit index:
```python
>>> def count(s, value):
        """Count the number of occurrences of value in sequence s."""
        total, index = 0, 0
        while index < len(s):
            if s[index] == value:
                total = total + 1
            index = index + 1
        return total
>>> count(digits, 8)
2
```
The `for` statement iterates over element values directly, without introducing `index` at all:
```python
>>> def count(s, value):
        """Count the number of occurrences of value in sequence s."""
        total = 0
        for elem in s:
            if elem == value:
                total = total + 1
        return total
>>> count(digits, 8)
2
```
Unpacking extends the same idea to sequences of pairs — counting pairs with equal elements:
```python
>>> pairs = [[1, 2], [2, 2], [2, 3], [4, 4]]
>>> same_count = 0
>>> for x, y in pairs:
        if x == y:
            same_count = same_count + 1
>>> same_count
2
```

### 2. Perfect numbers and minimum perimeter: map/filter/aggregate end to end
A perfect number equals the sum of its divisors (positive integers less than `n` dividing evenly). Divisors via comprehension:
```python
>>> def divisors(n):
        return [1] + [x for x in range(2, n) if n % x == 0]
>>> divisors(12)
[1, 2, 3, 4, 6]
>>> [n for n in range(1, 1000) if sum(divisors(n)) == n]
[6, 28, 496]
```
(The author notes 1 is typically also considered perfect, but doesn't qualify under this `divisors` definition.) Reuse `divisors` for a second problem — minimum perimeter of an integer-sided rectangle of given area. Height must divide the area; map every candidate height to a perimeter, then aggregate with `min`:
```python
>>> def width(area, height):
        assert area % height == 0
        return area // height
>>> def perimeter(width, height):
        return 2 * width + 2 * height
>>> def minimum_perimeter(area):
        heights = divisors(area)
        perimeters = [perimeter(width(area, h), h) for h in heights]
        return min(perimeters)
>>> minimum_perimeter(80)
36
```
The same problem in pure higher-order style — `keep_if` for filtering, `reduce` for aggregation:
```python
>>> def divisors_of(n):
        divides_n = lambda x: n % x == 0
        return [1] + keep_if(divides_n, range(2, n))
>>> from operator import add
>>> def sum_of_divisors(n):
        return reduce(add, divisors_of(n), 0)
>>> def perfect(n):
        return sum_of_divisors(n) == n
>>> keep_if(perfect, range(1, 1000))
[1, 6, 28, 496]
```
Note the result includes 1 here, unlike the comprehension version: `divisors_of(1)` is `[1]` (since `range(2, 1)` is empty), so `sum_of_divisors(1) == 1` and 1 passes `perfect`.

### 3. Trees: fib_tree, count_leaves, partition trees
The abstraction, with validation:
```python
>>> def tree(root_label, branches=[]):
        for branch in branches:
            assert is_tree(branch), 'branches must be trees'
        return [root_label] + list(branches)
>>> def label(tree):
        return tree[0]
>>> def branches(tree):
        return tree[1:]
>>> def is_tree(tree):
        if type(tree) != list or len(tree) < 1:
            return False
        for branch in branches(tree):
            if not is_tree(branch):
                return False
        return True
>>> def is_leaf(tree):
        return not branches(tree)
>>> t = tree(3, [tree(1), tree(2, [tree(1), tree(1)])])
>>> t
[3, [1], [2, [1], [1]]]
>>> label(branches(t)[1])
2
>>> is_leaf(branches(t)[0])
True
```
Tree-recursive **construction**: the nth Fibonacci tree has root label fib(n) and, for n > 1, two Fibonacci-tree branches:
```python
>>> def fib_tree(n):
        if n == 0 or n == 1:
            return tree(n)
        else:
            left, right = fib_tree(n-2), fib_tree(n-1)
            fib_n = label(left) + label(right)
            return tree(fib_n, [left, right])
>>> fib_tree(5)
[5, [2, [1], [1, [0], [1]]], [3, [1, [0], [1]], [2, [1], [1, [0], [1]]]]]
```
Tree-recursive **processing**: base case at the leaf, sum over branches:
```python
>>> def count_leaves(tree):
      if is_leaf(tree):
          return 1
      else:
          branch_counts = [count_leaves(b) for b in branches(tree)]
          return sum(branch_counts)
>>> count_leaves(fib_tree(5))
8
```
Partition trees encode the choice structure of integer partitioning; `print_parts` walks the tree accumulating a partition and prints it at each `True` leaf:
```python
>>> def partition_tree(n, m):
        """Return a partition tree of n using parts of up to m."""
        if n == 0:
            return tree(True)
        elif n < 0 or m == 0:
            return tree(False)
        else:
            left = partition_tree(n-m, m)
            right = partition_tree(n, m-1)
            return tree(m, [left, right])
>>> partition_tree(2, 2)
[2, [True], [1, [1, [True], [False]], [False]]]

>>> def print_parts(tree, partition=[]):
        if is_leaf(tree):
            if label(tree):
                print(' + '.join(partition))
        else:
            left, right = branches(tree)
            m = str(label(tree))
            print_parts(left, partition + [m])
            print_parts(right, partition)
>>> print_parts(partition_tree(6, 4))
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
Slicing applies to branches too — **binarization** groups adjacent branches to bound branch count at two:
```python
>>> def right_binarize(tree):
        """Construct a right-branching binary tree."""
        if is_leaf(tree):
            return tree
        if len(tree) > 2:
            tree = [tree[0], tree[1:]]
        return [right_binarize(b) for b in tree]
>>> right_binarize([1, 2, 3, 4, 5, 6, 7])
[1, [2, [3, [4, [5, [6, 7]]]]]]
```

### 4. Linked lists end to end: abstraction → sequence behaviors → partitions
Represent `[1, 2, 3, 4]` as `four = [1, [2, [3, [4, 'empty']]]]`. Define the abstract data representation:
```python
>>> empty = 'empty'
>>> def is_link(s):
        """s is a linked list if it is empty or a (first, rest) pair."""
        return s == empty or (len(s) == 2 and is_link(s[1]))
>>> def link(first, rest):
        """Construct a linked list from its first element and the rest."""
        assert is_link(rest), "rest must be a linked list."
        return [first, rest]
>>> def first(s):
        """Return the first element of a linked list s."""
        assert is_link(s), "first only applies to linked lists."
        assert s != empty, "empty linked list has no first element."
        return s[0]
>>> def rest(s):
        """Return the rest of the elements of a linked list s."""
        assert is_link(s), "rest only applies to linked lists."
        assert s != empty, "empty linked list has no rest."
        return s[1]
>>> four = link(1, link(2, link(3, link(4, empty))))
```
The pairs here are two-element lists — but pairs were also implementable as functions, so linked lists could be implemented using functions alone. Prove the sequence abstraction holds by implementing length and element selection, iteratively:
```python
>>> def len_link(s):
        """Return the length of linked list s."""
        length = 0
        while s != empty:
            s, length = rest(s), length + 1
        return length
>>> def getitem_link(s, i):
        """Return the element at index i of linked list s."""
        while i > 0:
            s, i = rest(s), i - 1
        return first(s)
>>> len_link(four), getitem_link(four, 1)
(4, 2)
```
and recursively:
```python
>>> def len_link_recursive(s):
        """Return the length of a linked list s."""
        if s == empty:
            return 0
        return 1 + len_link_recursive(rest(s))
>>> def getitem_link_recursive(s, i):
        """Return the element at index i of linked list s."""
        if i == 0:
            return first(s)
        return getitem_link_recursive(rest(s), i - 1)
```
(The book's environment-diagram walkthrough of `getitem_link(four, 1)` deliberately violates the abstraction barrier — defining `four` with raw list literals — to make the computational process inspectable; each `while` iteration rebinds `s` to a shorter suffix.) Recursive transformers and combiners:
```python
>>> def extend_link(s, t):
        """Return a list with the elements of s followed by those of t."""
        assert is_link(s) and is_link(t)
        if s == empty:
            return t
        else:
            return link(first(s), extend_link(rest(s), t))
>>> def apply_to_all_link(f, s):
        """Apply f to each element of s."""
        assert is_link(s)
        if s == empty:
            return s
        else:
            return link(f(first(s)), apply_to_all_link(f, rest(s)))
>>> def keep_if_link(f, s):
        """Return a list with elements of s for which f(e) is true."""
        assert is_link(s)
        if s == empty:
            return s
        else:
            kept = keep_if_link(f, rest(s))
            if f(first(s)):
                return link(first(s), kept)
            else:
                return kept
>>> def join_link(s, separator):
        """Return a string of all elements in s separated by separator."""
        if s == empty:
            return ""
        elif rest(s) == empty:
            return str(first(s))
        else:
            return str(first(s)) + separator + join_link(rest(s), separator)
>>> apply_to_all_link(lambda x: x*x, four)
[1, [4, [9, [16, 'empty']]]]
>>> keep_if_link(lambda x: x%2 == 0, four)
[2, [4, 'empty']]
>>> join_link(four, ", ")
'1, 2, 3, 4'
```
Culmination — **recursive construction**, where linked lists shine. Enumerate partitions explicitly with the same analysis as Chapter 1's `count_partitions`: partitioning `n` with parts up to `m` means either partitioning `n-m` with parts up to `m` (prepend `m` to each result) or partitioning `n` with parts up to `m-1`. Base cases: 0 has one empty partition; a negative `n` or `m == 0` has none.
```python
>>> def partitions(n, m):
        """Return a linked list of partitions of n using parts of up to m.

        Each partition is represented as a linked list.
        """
        if n == 0:
            return link(empty, empty)  # A list containing the empty partition
        elif n < 0 or m == 0:
            return empty
        else:
            using_m = partitions(n-m, m)
            with_m = apply_to_all_link(lambda s: link(m, s), using_m)
            without_m = partitions(n, m-1)
            return extend_link(with_m, without_m)

>>> def print_partitions(n, m):
        lists = partitions(n, m)
        strings = apply_to_all_link(lambda s: join_link(s, " + "), lists)
        print(join_link(strings, "\n"))
>>> print_partitions(6, 4)   # prints the same nine partitions as print_parts above
```
The result is highly nested — a linked list of linked lists, each made of nested pairs — and `join_link` with two different separators renders it readable.

## Common Errors
- **`in` on strings matches substrings, not elements.** The string abstraction does not conform to the full sequence abstraction: `'here' in "Where's Waldo?"` is `True`. The membership operator has an entirely different behavior on strings than on other sequences.
- **End-exclusive bounds.** Both `range(1, 10)` ("includes 1, but not 10") and slices take one-beyond-the-last as the second integer. Forgetting this gives off-by-one results.
- **Loop name leaks.** After `for <name> in ...`, the name remains bound to the last element in the current frame — the `for` statement updates the environment, it doesn't scope the name.
- **`\n` is one character.** In a multiline string, backslash-en appears as two characters but counts as a single element for length and element selection.
- **Over-enriching user-defined abstractions.** Rich abstractions are harder to implement for new types and take longer to learn; keep user-defined abstractions as simple as possible unless ubiquity justifies the cost.
- **Expecting cheap length/indexing from linked lists.** `len_link`/`getitem_link` walk the chain, taking time proportional to position; Python's built-in sequences are implemented differently and do not have a large cost for length or element retrieval.
- **Violating abstraction barriers for convenience.** The book flags its own violation (building `four` from raw literals for environment diagrams) as a deliberate, acknowledged trade-off for inspectability — not a pattern to copy into programs.
- **Confusing comprehension `for` with the `for` statement.** Inside square brackets, `for` is part of a list comprehension expression, not a statement.

## Key Takeaways
1. Program against the sequence abstraction (length + element selection), not against a specific type; membership and slicing are Python's extensions of it.
2. Decompose sequence problems into map, filter, and aggregate; list comprehensions are the idiomatic Python form, higher-order functions (`apply_to_all`/`keep_if`/`reduce`) the equivalent general form, and `map`/`filter`/`functools.reduce` the built-in names.
3. Closure (lists in lists) is what makes hierarchical data possible; trees impose regularity on that hierarchy via `tree`/`label`/`branches`/`is_leaf`, with `is_tree` enforcing well-formedness in the constructor.
4. Tree problems are solved by tree-recursive functions: base case at leaves, recursive combination over branches — for both constructing trees (`fib_tree`, `partition_tree`) and processing them (`count_leaves`, `print_parts`).
5. Linked lists show that a full sequence type can be built from pairs alone (hence from functions alone); their behavior condition is that constructor and selectors are inverses, and they shine for incremental recursive construction (`partitions`).
6. Keep user-defined abstractions simple; richness must be earned by ubiquity — sequences and strings earn it, most of your abstractions won't.

## Connects To
- **Section 2.2**: trees and linked lists are data abstractions built with constructors/selectors and behavior conditions; `getitem_link`'s diagram example knowingly crosses the abstraction barrier from 2.2.3.
- **Chapter 1**: `partitions` and `partition_tree` enumerate what `count_partitions` only counted, with the identical recursive decomposition.
- **Chapter 4**: the general definition of "iterable", and the lazy built-in `map`/`filter`.
- **Section 2.8**: the walking cost of linked-list length/indexing previews orders of growth.
- **Section 2.9**: linked lists and trees return as classes (`Link`, `Tree`) once OOP is available.
