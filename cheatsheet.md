# Cheatsheet — Composing Programs (decision rules)

## Choosing an abstraction mechanism

| Situation | Reach for | Not |
|---|---|---|
| Two functions share a skeleton | higher-order function | copy-paste |
| Compound value, few operations | constructor/selector functions | a class |
| State tied to identity, many instances | class | module-level globals |
| One-off stateful behavior | closure + `nonlocal` | a single-instance class |
| Operation across types | shared interface (`__len__` etc.) → type dispatch → coercion, in that order | isinstance ladders everywhere |
| Domain outgrows the language | small interpreter / DSL | ever-deeper config dicts |

## Recursion decisions

- Write the **base case first**; if you can't state it, you don't understand the decomposition.
- Verify one level only (**recursive leap of faith**); tracing full call trees is a smell.
- Function calls itself twice per invocation → **tree recursion** → expect exponential time → memoize if pure.
- Need previous-two values only → iteration beats recursion in Python (no tail-call optimization).

## Mutability rules

- Default to **pure functions**; introduce mutation only where identity matters.
- `a is b` ≠ `a == b` the moment data is mutable; test identity for object semantics, equality for values.
- Never use a mutable default argument — it is created once at `def`.
- Mutation inside a function that also returns values = the book's definition of a non-pure function; document it.

## Performance thresholds

| Symptom | Diagnosis | Fix |
|---|---|---|
| Same pure function recomputed on repeat args | tree recursion overlap | memoize: Θ(φⁿ)→Θ(n) |
| Sequence built fully but consumed partially | eager evaluation | generator / stream |
| Space blow-up in recursion | counting all frames, but only **active** environments occupy memory | convert to iteration if depth is the issue |
| Nested loops over same data | Θ(n²) | sort/index first, or use a dict |

## Tells & smells

- Reading a function's body to use it → its docstring/abstraction failed (fix the interface).
- Code reaches past constructors/selectors into representation → **abstraction barrier violation**; will cost at the next representation change.
- A class with one instance and no identity semantics → should be a closure or module.
- `except:` bare → you will swallow the error you need; catch the specific class.
- Growing chain of `isinstance` → missing interface or dispatch table.

## Interpreter quick path

1. Tokenize → expression tree (Pairs).
2. `eval`: dispatch on form — primitive? name? call expression? special form?
3. `apply`: primitive → call it; user-defined → new frame bound to formals, eval body.
4. Wrap in REPL: `try/except` keeps the loop alive on user errors.
