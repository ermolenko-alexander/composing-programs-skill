# Section 3.2: Functional Programming

## Core Idea
A small Scheme subset — expressions only, no statements, no mutation, symbolic data — uses essentially Python's model of computation yet is a rich, fully functional language; learning it supplies the object language our interpreters will implement.

## Frameworks Introduced
- **Call expressions vs special forms**: every Scheme program is expressions; a form that looks like a call but has its own evaluation procedure is a **special form** (`if`, `define`, `lambda`, `and`, `or`, `begin`).
  - When to use: classifying any parenthesized form before evaluating it.
  - How: call expression → evaluate operator and all operands, then apply; special form → follow its dedicated rule (e.g. `if` evaluates predicate first, then only one branch).
- **Quotation**: `'expr` refers to the symbol or list itself rather than its value — "quotation allows us to talk about language itself".
  - When to use: constructing symbolic data, or representing code as data.
  - How: `(list 'a 'b)` gives `(a b)` regardless of what a and b are bound to; `'(a b c)` types in a compound object directly.
- **Pairs and recursive lists**: `cons`/`car`/`cdr` with `nil` (`'()`) build all compound data.
  - When to use: any sequence or tree in Scheme.
  - How: list = pair whose cdr is a list or nil; process with recursion on `null?`.

## Key Concepts
- **Prefix notation**: operator first, always parenthesized: `(+ (* 3 5) (- 10 6))`.
- **Special form**: syntactically call-like expression with a different evaluation procedure.
- **Procedure**: Scheme's term for a function; defined with `(define (<name> <formal parameters>) <body>)`.
- **lambda**: `(lambda (<formal-parameters>) <body>)` — same procedure as `define` makes, just unnamed; `(define (plus4 x) (+ x 4))` ≡ `(define plus4 (lambda (x) (+ x 4)))`.
- **Short-circuit booleans**: `and` returns first false value (else last value); `or` returns first true value; later operands are not evaluated.
- **cons / car / cdr / nil**: pair constructor, first element, rest, empty list.
- **null?**: predicate for the empty list; the base case of list recursion.
- **Quoted expression**: any expression that is not evaluated; symbols as data.
- **begin**: special form combining multiple sub-expressions into one (used for sequencing turtle commands).

## Mental Models
- Think of Scheme as Python with only expressions: the call-expression evaluation rule is identical (evaluate operator and operands, apply).
- Think of quotation as the use/mention distinction from philosophy: the dog vs the word "dog".
- Use nested `define` with recursion where Python would use a loop — iteration is recursion here (`sqrt-iter`).
- Lexical scoping works as in Python: inner definitions see enclosing formals.

## Anti-patterns
- **Evaluating a special form like a call**: `(if p c a)` must not evaluate both branches; `and`/`or` must short-circuit.
- **`(list a b)` when you mean symbols**: it builds a list of values; use `'a 'b` for the symbols themselves.
- **Reaching for mutation**: this subset has none — express state change through recursion and new values.

## Code Examples
```scheme
(define (sqrt x)
  (define (good-enough? guess)
    (< (abs (- (square guess) x)) 0.001))
  (define (improve guess)
    (average guess (/ x guess)))
  (define (sqrt-iter guess)
    (if (good-enough? guess)
        guess
        (sqrt-iter (improve guess))))
  (sqrt-iter 1.0))
```
- **What it demonstrates**: local definitions, lexical scope, and iteration-as-recursion in pure expressions.

## Worked Example
Sierpinski's triangle via turtle graphics: `(repeat k fn)` recurses with `begin` to run a thunk k times; `(tri fn)` repeats a drawing thunk 3 times with `(lt 120)` after each; `(sier d k)` draws a plain triangle (`fd d`) at depth 1, else a triangle of `(leg d k)` calls; `leg` recursively calls `(sier (/ d 2) (- k 1))` for the first half of the leg, then `penup`/`fd d`/`pendown` to reach the next vertex. The mutual recursion between `sier` and `leg` yields the full fractal with `(sier 400 6)` — a complete recursive drawing in four tiny definitions.

## Key Takeaways
1. Classify every form first: call expression or special form — the evaluation rule differs.
2. Lists are nested pairs ending in nil; write list code as recursion with `null?` base case (`length`, `getitem`).
3. Quotation turns code and symbols into data — the prerequisite for interpreters that read programs as lists.
4. `define`-with-name is sugar for binding a `lambda`; procedures are first-class values.
5. This immutable expression-only subset is enough to express most ideas from the whole text.

## Connects To
- **Section 3.4**: Calculator borrows Scheme's call-expression syntax; the interpreter represents these programs as `Pair` lists.
- **Section 3.5**: special forms (`define`, `lambda`, `if`) get their evaluation rules implemented in `scheme_eval`.
- **Section 3.1**: Scheme is the minimal powerful language promised there.
