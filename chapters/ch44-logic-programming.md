# Section 4.4: Logic Programming

## Core Idea
In the `logic` language (a Prolog-like declarative query language built on the Scheme interpreter), programs are declared facts and queries over relations; the query interpreter retrieves matching facts and deduces new ones by logical inference — the user states what must hold, never how to find it.

## Frameworks Introduced
- **Facts and queries**: a `fact` statement declares a relation (not a procedure application); a `query` contains variables (symbols starting with `?`) that the interpreter matches against facts.
  - How: `(fact (parent abraham barack))`; `(query (parent abraham ?child))` → Success! + substitutions for `?child`.
- **Compound facts (rules)**: `(fact <conclusion> <hypothesis0> ... <hypothesisN>)` — the conclusion holds when all hypotheses are satisfied. Relation types need no advance definition.
- **Negation as failure**: `(query (not <relation>))` succeeds iff the relation fails.
- **Recursive facts**: a fact's conclusion may depend on a hypothesis using the same relation symbol — e.g. the two-fact ancestor definition; chains of inference prove derived facts.
- **Compound queries**: multiple subexpressions must be satisfied simultaneously by one assignment; a repeated variable must take the same value everywhere.
- **Hierarchical facts and dot notation**: facts/queries may contain nested lists; a variable after a dot matches the rest of a list (`. ?details`), enabling variable-length structures like pedigrees.

## Key Concepts
- **Fact**: a declared relation stored in the database.
- **Relation**: a named association among values, matched (never applied) to queries.
- **Query**: one or more lists after `query`; may verify a fact (no variables) or bind variables.
- **Variable**: a symbol beginning with `?`, bound by matching.
- **Conclusion / hypotheses**: the head and body of a compound fact.
- **Negation as failure**: not(R) is true exactly when R cannot be proven.
- **Dotted list**: `(?a . ?r)` splits a list into first and rest inside facts or queries.
- **Inference chain**: a derived fact proved by proving each supporting fact in succession.

## Mental Models
- Read a compound fact as "conclusion, provided that hypotheses": `(fact (child ?c ?p) (parent ?p ?c))` = "?c is the child of ?p provided ?p is the parent of ?c."
- Think of relations as reversible: `append-to-form` both appends two lists and enumerates all splits of a list — the same two facts answer both directions.
- A query with several clauses is a join: shared variables play the role of SQL's equality `where` conditions.

## Anti-patterns
- **Expecting classical negation**: `(query (not (parent abraham ?who)))` fails even though many bindings "should" work — the inner relation succeeds for some binding, so its negation fails. Negation as failure tests provability, not per-binding truth.
- **Thinking the interpreter is intelligent**: all deduction reduces to one operation repeated many times — matching two lists containing variables in an environment (unification, Section 4.5).

## Code Examples
```scheme
(fact (ancestor ?a ?y) (parent ?a ?y))
(fact (ancestor ?a ?y) (parent ?a ?z) (ancestor ?z ?y))

(query (ancestor ?a herbert))

(fact (append-to-form () ?x ?x))
(fact (append-to-form (?a . ?r) ?y (?a . ?z)) (append-to-form ?r ?y ?z))
(query (append-to-form ?left ?right (a b c d e)))
```
- **What it demonstrates**: recursive relation definition (base + recursive fact) and the bidirectional power of relational `append`.

## Worked Example
Dog genealogy. Simple facts record parentage (`(fact (parent fillmore delano))`, etc.). `ancestor` is defined by two recursive facts. Proving `(ancestor fillmore herbert)` chains: (1) `(parent delano herbert)` simple fact → (2) `(ancestor delano herbert)` by the 1st ancestor fact → (3) `(parent fillmore delano)` simple fact → (4) `(ancestor fillmore herbert)` by the 2nd ancestor fact. Hierarchical `dog` facts add `(name ...)` and `(color ...)`; a three-clause compound query joins them to find ancestor–descendant pairs sharing a color. Dotted facts build pedigrees: `(query (pedigree barack . ?lineage))` lists barack's whole chain of ancestry.

## Key Takeaways
1. Declare relations as facts; let the interpreter prove queries — no control flow to write.
2. Compound facts are inference rules: conclusion + hypotheses.
3. Recursive facts let two lines define unbounded relations (ancestor, append).
4. Shared variables across clauses express joins and equality constraints.
5. `not` is negation as failure — it checks whether a relation is unprovable, which surprises on queries with free variables.
6. Dot notation matches list tails, enabling variable-arity facts.

## Connects To
- **Section 4.5**: unification is the single matching operation that powers all of this inference.
- **Section 4.3**: facts ↔ table rows, compound queries ↔ joins; logic generalizes SQL's declarative stance.
- **Chapter 3 (Scheme interpreter)**: the logic interpreter is built on it; facts and queries are Scheme lists.
