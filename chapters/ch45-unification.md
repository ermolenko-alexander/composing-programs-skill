# Section 4.5: Unification

## Core Idea
The query interpreter's entire power reduces to one primitive — unification, matching two expressions that may both contain variables — applied repeatedly by a depth-limited recursive search over facts. The logic language is a prover: facts are axioms, queries are assertions to establish.

## Frameworks Introduced
- **Pattern matching**: a pattern matches an expression (a possibly nested Scheme list) if some binding of variables to values makes substitution into the pattern yield the expression. E.g. `((a b) c (a b))` matches `(?x c ?x)` with `?x` bound to `(a b)`.
- **The unification algorithm**: generalizes pattern matching to two expressions that both contain variables. Recursive conditions, in order:
  1. Replace `e` and `f` by their values if they are bound variables (lookup).
  2. If `e == f`, succeed.
  3. If `e` is a variable, bind `e` to `f` and succeed; symmetrically for `f`.
  4. If neither is a variable and either is an atom (and they are unequal), fail.
  5. Otherwise both are pairs: unify first with first and second with second.
- **Proofs**: each fact is an axiom; a query asserts there exists an assignment making all sub-expressions follow from the axioms; each returned assignment is a trace of a proof.
- **Search procedure**: the recursive, generator-based `search(clauses, env, depth)` chooses which expressions to unify, chaining facts; it yields environments of successful bindings and limits nesting by `DEPTH_LIMIT`.
- **Unique names (rename_variables)**: before unifying with a fact, rename its variables with a unique integer suffix (`?x` → `?x_7`) so an `?x` in one fact is never confused with `?x` in another.

## Key Concepts
- **Unification**: the general method of matching a query to a fact where both may contain variables.
- **Frame / env**: an environment of variable bindings; unification "destructively extends" it, but existing bindings are never changed.
- **bind**: recursively substitutes bound variables into an expression until none remain (e.g. producing `((a b c) (a b c))`).
- **Ground**: a clause with its variables substituted by current env values (used for negation).
- **Negation as failure in search**: for a `not` clause, run a sub-search; if it yields nothing, continue with remaining clauses; if it yields anything, fail immediately.
- **DEPTH_LIMIT**: a cap on the nested application of rules, keeping search finite.
- **Generator-based search**: `search` is a Python generator; each `yield env` is one solution, so results stream lazily.

## Mental Models
- Think of unification as two-way pattern matching: either side may contribute the variables, and a variable can bind to an expression that itself contains variables (`?x` → `(a ?y c)`), resolved later by `bind`.
- Think of the interpreter's apparent "intelligence" as one simple operation repeated: unify query clause with a fact's conclusion, then recursively establish the fact's hypotheses, then the rest of the query clauses.
- Think of each successful search result as a proof trace in a formal system.

## Anti-patterns
- **Sharing variable names across facts**: unification assumes no variable is shared between the two expressions; without `rename_variables`, unrelated `?x`s collide.
- **Unbounded recursive rules**: recursive facts can chain forever; the depth limit is what keeps the solver from diverging (and also limits what it can prove).

## Code Examples
```python
def unify(e, f, env):
    """Destructively extend ENV so as to unify (make equal) e and f."""
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
- **What it demonstrates**: the complete unification algorithm — lookup, equality, variable binding, atom failure, recursive descent on pairs.

## Worked Example
Unifying `(?x ?x)` with `((a ?y c) (a b ?z))`. Step 1: match first elements — bind `?x` to `(a ?y c)`. Step 2: match second elements — replace `?x` by its value, then unify `(a ?y c)` with `(a b ?z)`, binding `?y`→`b` and `?z`→`c`. The env now holds all three bindings, and `bind(e, env)` prints `((a b c) (a b c))` — the variable-free expression that matches both patterns. On top of this, `search` proves compound queries: it renames a fact's variables, unifies the fact's conclusion with the query's first clause in a child Frame, recursively establishes the fact's hypotheses, then recursively establishes the remaining query clauses, yielding every environment that succeeds — e.g. finding a common ancestor of clinton and a brown dog from the dog-genealogy facts.

## Key Takeaways
1. Unification = pattern matching where both sides may hold variables; five ordered cases implement it.
2. Bindings may be partial (variable → expression with variables); `bind` resolves them transitively.
3. Inference = unify conclusion, then recursively establish hypotheses, then remaining clauses — a nested generator search.
4. Rename fact variables with unique ids before each unification.
5. Negation is implemented as a failing sub-search; it grounds the clause first.
6. A depth limit makes the general problem solver terminate, at the cost of completeness; efficient inference remains an open research topic.

## Connects To
- **Section 4.4**: implements the logic language whose facts/queries it proves.
- **Section 4.2**: `search` is a generator — solutions are produced lazily via `yield`.
- **Chapter 3**: expressions are `Pair`/`nil` Scheme lists and `Frame` environments from the Scheme interpreter.
