# Section 4.5: Unification

## Core Idea
This section implements the query interpreter that performs inference in the `logic` language. The interpreter is a **general problem solver**, but with substantial limitations on the scale and type of problems it can solve; the construction of efficient inference procedures remains an active research topic. The fundamental operation is **unification**: a general method of matching a query to a fact, each of which may contain variables. The interpreter applies this operation repeatedly — first to match the original query to conclusions of facts, then to match the hypotheses of facts to other conclusions in the database — performing a search through the space of all facts related to a query. If it finds a way to support the query with an assignment of values to variables, it returns that assignment as a successful result.

## Formal Rules

### Pattern matching
A pattern matches some expression (a possibly nested Scheme list) if there is a binding of variable names to values such that substituting those values into the pattern yields the expression. E.g. `((a b) c (a b))` matches `(?x c ?x)` with `?x` bound to `(a b)`; the same expression matches `((a ?y) ?z (a b))` with `?y` → `b`, `?z` → `c`.

### The unification algorithm — conditions checked in order
Unification generalizes pattern matching to two expressions that may *both* contain variables, recursing on corresponding parts until a contradiction is reached or a viable binding to all variables is established:
1. Both inputs `e` and `f` are replaced by their values if they are (bound) variables.
2. If `e` and `f` are equal, unification succeeds.
3. If `e` is a variable, unification succeeds and `e` is bound to `f`.
4. If `f` is a variable, unification succeeds and `f` is bound to `e`.
5. If neither is a variable, both are not lists, and they are not equal, then `e` and `f` cannot be unified — failure.
6. Otherwise both are pairs: unification is performed on both their first and second corresponding elements.

```python
>>> def unify(e, f, env):
        """Destructively extend ENV so as to unify (make equal) e and f, returning
        True if this succeeds and False otherwise.  ENV may be modified in either
        case (its existing bindings are never changed)."""
        e = lookup(e, env)
        f = lookup(f, env)
        if e == f:
            return True
        elif isvar(e):
            env.define(e, f)
            return True
        elif isvar(f):
            env.define(f, e)
            return True
        elif scheme_atomp(e) or scheme_atomp(f):
            return False
        else:
            return unify(e.first, f.first, env) and unify(e.second, f.second, env)
```

### Representation
Facts and queries are Scheme lists using the `Pair` class and `nil` object from the previous chapter: `read_line("(?x c ?x)")` → `Pair('?x', Pair('c', Pair('?x', nil)))`. An environment binding symbols to values is a `Frame` instance with a `bindings` attribute. Replicable via `from logic import *`.

### Unique names
Unification assumes no variable is shared between `e` and `f`. But facts and queries reuse names — an `?x` in one fact is unrelated to `?x` in another. Before a fact is passed to `unify`, its variable names are replaced by unique names via `rename_variables`, appending a unique integer for the fact:
```python
>>> def rename_variables(expr, n):
        """Rename all variables in EXPR with an identifier N."""
        if isvar(expr):
            return expr + '_' + str(n)
        elif scheme_pairp(expr):
            return Pair(rename_variables(expr.first, n),
                        rename_variables(expr.second, n))
        else:
            return expr
```

## Worked Examples

### 1. Basic pattern match with `unify`
```python
>>> e = read_line("((a b) c (a b))")
>>> f = read_line("(?x c ?x)")
>>> env = Frame(None)
>>> unify(e, f, env)
True
>>> env.bindings
{'?x': Pair('a', Pair('b', nil))}
>>> print(env.lookup('?x'))
(a b)
```
True means the pattern `f` matched the expression `e`; the result is recorded as the binding of `?x` to `(a b)`.

### 2. Two-sided unification trace: `(?x ?x)` vs `((a ?y c) (a b ?z))`
These unify because an expression with no variables matches both: `((a b c) (a b c))`. Steps:
1. To match the first elements, bind `?x` to `(a ?y c)`.
2. To match the second elements, first replace `?x` by its value; then match `(a ?y c)` to `(a b ?z)` by binding `?y` → `b` and `?z` → `c`.
```python
>>> e = read_line("(?x ?x)")
>>> f = read_line("((a ?y c) (a b ?z))")
>>> env = Frame(None)
>>> unify(e, f, env)
True
>>> env.bindings
{'?z': 'c', '?y': 'b', '?x': Pair('a', Pair('?y', Pair('c', nil)))}
>>> print(bind(e, env))
((a b c) (a b c))
```
Note `?x` is bound to an expression that **itself contains a variable**. The `bind` function recursively and repeatedly substitutes all variables by their values in an expression until no bound variables remain.

### 3. Proof as a query result
"One way to think about the `logic` language is as a prover of assertions in a formal system. Each stated fact establishes an axiom, and each query must be established by the query interpreter from these axioms" — each query asserts there is some assignment to its variables such that all sub-expressions simultaneously follow from the facts. Given the dog-genealogy facts (parents, the two `ancestor` rules, and `dog`/`color` records), assert there is a common ancestor of clinton and a brown dog:
```scheme
(query (ancestor ?a clinton)
       (ancestor ?a ?brown-dog)
       (dog (name ?brown-dog) (color brown)))
```
The interpreter outputs `Success!` only if it can establish the assertion; each returned assignment is a **trace of a larger proof** (a full proof would include all facts used, e.g. `(parent abraham clinton)` and `(parent fillmore abraham)`).

## The Search Procedure
Unification is the primitive; the **search procedure** chooses what expressions to unify so that a set of facts chains together to establish the query. `search` takes the Scheme list of query `clauses`, an environment `env` of current bindings (initially empty), and the `depth` of rules chained so far:

```python
>>> def search(clauses, env, depth):
        """Search for an application of rules to establish all the CLAUSES,
        non-destructively extending the unifier ENV.  Limit the search to
        the nested application of DEPTH rules."""
        if clauses is nil:
            yield env
        elif DEPTH_LIMIT is None or depth <= DEPTH_LIMIT:
            if clauses.first.first in ('not', '~'):
                clause = ground(clauses.first.second, env)
                try:
                    next(search(clause, glob, 0))
                except StopIteration:
                    env_head = Frame(env)
                    for result in search(clauses.second, env_head, depth+1):
                        yield result
            else:
                for fact in facts:
                    fact = rename_variables(fact, get_unique_id())
                    env_head = Frame(env)
                    if unify(fact.first, clauses.first, env_head):
                        for env_rule in search(fact.second, env_head, depth+1):
                            for result in search(clauses.second, env_rule, depth+1):
                                yield result
```

Execution logic, as the text explains it:
- Empty clauses → yield the environment: a solution.
- **Negated first clause**: rather than unifying it with a fact, check that *no* unification is possible via a recursive `search` on the grounded clause (`ground` substitutes current env values). If that recursive call yields nothing (`StopIteration`), continue with the rest of the clauses; if unification is possible, fail immediately.
- **Ordinary first clause**: for each fact in the database, rename its variables with a unique id, then attempt to unify the fact's conclusion (`fact.first`) with the query's first clause in a **new child environment** `env_head` (bindings happen as a side effect). On success:
  - the middle `for` establishes the **hypotheses of the rule** (`fact.second`) — this is where a recursive rule's hypotheses are passed recursively to `search`;
  - for every environment `env_rule` that establishes them, the final `for` searches to establish the **rest of the query's clauses**; every success is yielded.
- `DEPTH_LIMIT` caps the nested application of rules, keeping the search finite.

## Key Concepts
- **Unification**: matching two expressions that may both contain variables.
- **Frame / env**: binding environment; `unify` "destructively extends" it, but **existing bindings are never changed** (env may be modified even on failure).
- **lookup**: replace a bound variable by its value before comparing.
- **bind**: substitute variables transitively until none remain.
- **ground**: a clause with variables replaced by current env values (used for negation).
- **rename_variables / get_unique_id**: `?x` → `?x_7`, per fact, so unrelated variables never collide.
- **search**: a Python generator — each `yield env` streams one solution lazily.
- **DEPTH_LIMIT**: termination at the price of completeness.
- **Proof trace**: each successful assignment witnesses a proof of the query from the axioms.

## Mental Models
- Unification is two-way pattern matching: either side may contribute variables, and a variable may bind to an expression containing variables, resolved later by `bind`.
- The interpreter's apparent intelligence is one operation repeated: unify a query clause with a fact's conclusion, recursively establish the fact's hypotheses, then the remaining query clauses.
- The nested `for`/`yield` structure is a lazy depth-first search over proof trees; child `Frame`s make it non-destructive per branch.

## Anti-patterns / Common Errors
- **Sharing variable names across facts**: without `rename_variables`, an `?x` in one fact gets confused with the unrelated `?x` in another — bindings collide and wrong proofs appear.
- **Unbounded recursive rules**: recursive facts can chain forever; the depth limit is what keeps the solver from diverging, and also limits what it can prove.
- **Assuming env is untouched on failure**: `unify` may modify the environment either way — only *existing* bindings are safe; this is why each attempt gets a fresh child `Frame(env)`.
- **Expecting full-strength logic**: this is a simple general problem solver; efficient inference is open research, and the implementation trades completeness for termination.

## Key Takeaways
1. Unification = pattern matching where both sides may hold variables; six ordered conditions implement it (lookup, equality, bind-e, bind-f, atom failure, recurse on pairs).
2. Bindings may be partial (variable → expression with variables); `bind` resolves them transitively.
3. Inference = unify conclusion, then recursively establish hypotheses, then remaining clauses — a nested generator search over child frames.
4. Rename fact variables with unique integer suffixes before each unification.
5. Negation = a failing grounded sub-search; success of the inner search fails the `not` immediately.
6. A depth limit makes the solver terminate at the cost of completeness.
7. A query result is a proof trace: facts are axioms, the interpreter is a prover.

## Connects To
- **Section 4.4**: implements the logic language whose facts and queries it proves (dog genealogy reused as the axiom set).
- **Section 4.2**: `search` is a generator — solutions stream lazily via `yield`; `StopIteration` signals a failed sub-search.
- **Chapter 3**: expressions are `Pair`/`nil` Scheme lists; environments are `Frame`s from the Scheme interpreter project.
