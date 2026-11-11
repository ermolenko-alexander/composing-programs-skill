# Section 4.4: Logic Programming

## Core Idea
The `logic` language, designed for this text, is a declarative query language based upon Prolog and the declarative language in SICP. Data records are expressed as Scheme lists, and queries are expressed as Scheme values; the `logic` interpreter is a complete implementation that depends upon the Scheme project of the previous chapter. Databases store records representing facts; the purpose of the query interpreter is to retrieve collections of facts drawn directly from database records **and to deduce new facts from the database using logical inference**. The user declares what must hold, never how to find it — "the user of the language does not need to know how this information is combined, but only that the result has a particular form."

## Formal Rules

### Facts
- A `fact` statement consists of one or more lists following the keyword `fact`. A **simple fact** is a single list: `(fact (parent abraham barack))` declares "the dog Abraham is the parent of Barack."
- A fact is **not a procedure application** but a declared **relation**. Relation types do not need to be defined in advance. Relations are not applied, but *matched* to queries.
- **Compound fact (rule)**: `(fact <conclusion> <hypothesis0> <hypothesis1> ... <hypothesisN>)` — for the conclusion to be true, all of the hypotheses must be satisfied.
- **Recursive fact**: the conclusion may depend upon a hypothesis containing the same relation symbol. A single fact can imply a large number of additional facts, or even infinitely many, as long as the query interpreter can discover them.

### Queries
- A query consists of one or more lists following the keyword `query`. It may contain **variables**: symbols beginning with `?`, matched to facts by the interpreter. The interpreter responds `Success!` if the query matches some fact, then lists variable substitutions; a query that matches nothing returns failure.
- A query need not include variables — it may simply verify a fact: `(query (child herbert delano))`.
- **Compound query**: multiple subexpressions; all must be satisfied simultaneously by one assignment of symbols to variables. A variable appearing more than once must take the same value in each context.

### Negation as failure
`(query (not <relation>))` succeeds if `<relation>` fails, and fails if `<relation>` succeeds. The procedure: first inspect the inner relation; if it succeeds *for any binding*, the negation fails.

### Hierarchical facts and dot notation
Fact and query lists can contain lists, representing hierarchical data. Queries can articulate the full structure or match variables to whole sublists. A variable following a dot matches **the rest of the list**; dotted lists can appear in either facts or queries.

## Code Examples

### The dog genealogy database
```scheme
(fact (parent abraham barack))
(fact (parent abraham clinton))
(fact (parent delano herbert))
(fact (parent fillmore abraham))
(fact (parent fillmore delano))
(fact (parent fillmore grover))
(fact (parent eisenhower fillmore))

(query (parent abraham ?child))        ; Success! ?child: barack / clinton
```

### Compound fact and queries against it
```scheme
(fact (child ?c ?p) (parent ?p ?c))
; "?c is the child of ?p, provided that ?p is the parent of ?c."

(query (child ?child fillmore))        ; interpreter proves (child abraham fillmore) etc.
(query (child herbert delano))         ; verification, no variables
(query (child eisenhower ?parent))     ; failure — nothing matches
```

### Negation
```scheme
(query (not (parent abraham clinton)))   ; fails  — the inner relation succeeds
(query (not (parent abraham barack)))    ; fails  — likewise
(query (not (parent abraham ?who)))      ; FAILS — see Common Errors
```

### Recursive ancestor relation
```scheme
(fact (ancestor ?a ?y) (parent ?a ?y))
(fact (ancestor ?a ?y) (parent ?a ?z) (ancestor ?z ?y))
; ?a is an ancestor of ?y if it is a parent of ?y,
; or if it is the parent of an ancestor of ?y.

(query (ancestor ?a herbert))                      ; all ancestors of herbert
(query (ancestor ?a barack) (ancestor ?a herbert)) ; common ancestors (compound query)
```

### Hierarchical facts
```scheme
(fact (dog (name abraham) (color white)))
(fact (dog (name barack) (color tan)))
(fact (dog (name clinton) (color white)))
(fact (dog (name delano) (color white)))
(fact (dog (name eisenhower) (color tan)))
(fact (dog (name fillmore) (color brown)))
(fact (dog (name grover) (color tan)))
(fact (dog (name herbert) (color brown)))

(query (dog (name clinton) (color ?color)))   ; bind a leaf
(query (dog (name clinton) ?info))            ; bind a whole sublist
```

### Pedigrees with dot notation
```scheme
(fact (pedigree ?name) (dog (name ?name) . ?details))
(fact (pedigree ?child ?parent . ?rest)
      (parent ?parent ?child)
      (pedigree ?parent . ?rest))

(query (pedigree barack . ?lineage))   ; barack's whole chain of ancestry
```

### Relational append
```scheme
(fact (append-to-form () ?x ?x))
(fact (append-to-form (?a . ?r) ?y (?a . ?z)) (append-to-form ?r ?y ?z))

(query (append-to-form (a b c) (d e) ?result))         ; ?result: (a b c d e)
(query (append-to-form ?left ?right (a b c d e)))      ; ALL splits of the list
```
Base case: appending an empty list to any list gives that list. Recursive case: a list with first `?a` and rest `?r` appends to `?y` to form a list with first `?a` and rest `?z`, provided `?r` and `?y` append to form `?z`.

## Worked Examples

### 1. Inference chain for `(ancestor fillmore herbert)`
Recursive facts may require long chains of inference. To prove this fact, prove each of the following in succession:
```scheme
(parent delano herbert)       ; (1), a simple fact
(ancestor delano herbert)     ; (2), from (1) and the 1st ancestor fact
(parent fillmore delano)      ; (3), a simple fact
(ancestor fillmore herbert)   ; (4), from (2), (3), & the 2nd ancestor fact
```

### 2. Joining multiple kinds of facts
"Much of the power of a database lies in the ability of the query interpreter to join together multiple kinds of facts in a single query." All ancestor–descendant pairs sharing a color:
```scheme
(query (dog (name ?name) (color ?color))
       (ancestor ?ancestor ?name)
       (dog (name ?ancestor) (color ?color)))
```
The repeated `?color` forces equality across the two `dog` clauses; the repeated `?name`/`?ancestor` link all three clauses — exactly the role of SQL's `where` equalities in a three-way join.

### 3. Bidirectional append
The same two `append-to-form` facts answer both directions: given `(a b c)` and `(d e)`, deduce `(a b c d e)`; given `(a b c d e)`, enumerate all `?left`/`?right` pairs that append to form it. No separate "split" program is needed — relations have no fixed inputs and outputs.

## Key Concepts
- **Fact**: a declared relation stored in the database (simple or compound).
- **Relation**: a named association among values, matched (never applied) to queries.
- **Query**: lists after `query`; verifies or binds.
- **Variable**: symbol starting with `?`.
- **Conclusion / hypotheses**: head and body of a compound fact.
- **Recursive fact**: conclusion depends on a hypothesis with the same symbols.
- **Compound query**: conjunction; shared variables must agree.
- **Negation as failure**: `not R` ⇔ R is unprovable.
- **Hierarchical fact**: nested lists inside facts/queries.
- **Dotted variable**: `. ?rest` matches a list's tail; works in facts and queries.
- **Inference chain**: a derived fact proved by proving supporting facts in succession.

## Mental Models
- Read a compound fact as "conclusion, provided that hypotheses."
- Relations are reversible: any argument position can be the "output".
- A compound query is a join; shared variables are the join keys.
- The interpreter is not intelligent: it finds all of this "through one simple operation repeated many times: that of matching two lists that contain variables in an environment" (unification, 4.5).

## Anti-patterns / Common Errors
- **Expecting classical negation**: `(query (not (parent abraham ?who)))` fails even though many symbols could bind `?who`. The steps for negation first inspect `(parent abraham ?who)`; it succeeds (`?who` = barack or clinton), so its negation must fail. Negation as failure tests provability of the relation as a whole, not truth per binding.
- **Writing facts as if they were procedure calls**: relations are matched, not applied; there is no evaluation of "arguments".
- **Forgetting the base case of a recursive relation** (`append-to-form () ?x ?x`, the first `ancestor` fact): without it no chain of inference can bottom out.

## Key Takeaways
1. Declare relations as facts; the interpreter proves queries — no control flow to write.
2. Compound facts are inference rules: conclusion + hypotheses, all hypotheses required.
3. Two recursive facts suffice for unbounded relations (ancestor, append).
4. Shared variables across clauses express joins and equality constraints.
5. `not` is negation as failure and surprises on queries with free variables.
6. Dot notation matches list tails, enabling variable-arity facts (pedigree).
7. Relations run in any direction — append also splits.

## Connects To
- **Section 4.5**: unification is the single matching operation powering all of this; search composes it into proofs.
- **Section 4.3**: facts ↔ rows, compound queries ↔ joins; logic generalizes SQL's declarative stance (and adds recursion over structures, not just numbers).
- **Chapter 3 (Scheme interpreter)**: the logic interpreter is built on it; facts and queries are Scheme lists.
