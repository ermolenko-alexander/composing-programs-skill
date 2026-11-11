# Section 3.2: Functional Programming

## Core Idea
A small Scheme subset — expressions only (no statements), no mutable values, specializing in symbolic computation — employs a model of computation very similar to Python's, yet is a rich, fully functional language. Learning it supplies the object language the interpreters of 3.4–3.5 will implement.

## Context
Software spans physical languages (machine languages concerned with bits and primitive instructions) and high-level languages erected on that substrate, which hide bit-level representation and provide means of combination and abstraction appropriate to large-scale software organization. Scheme is a dialect of Lisp, the second-oldest language still widely used (after Fortran); modern Lisp dialects such as Clojure have fast-growing communities.

## Frameworks Introduced
- **Call expressions vs special forms**: every Scheme program consists of expressions, which are either call expressions or special forms. A special form looks syntactically like a call expression but has a different evaluation procedure.
  - When to use: classify any parenthesized form before evaluating it — this is the first dispatch in every Scheme interpreter.
- **Quotation**: `'expr` refers to the symbol or list itself rather than its value — quotation lets a language talk about language itself.
  - When to use: constructing symbolic data, or representing code as data (the prerequisite of 3.5's "data as programs").
- **Pairs and recursive lists**: `cons`/`car`/`cdr` with `nil` (`'()`) build all compound data.
  - How: a list is a pair whose cdr is a list or nil; process by recursion with a `null?` base case.

## Evaluation Rules (the author's formal procedures)
- **Call expression** `(op e1 ... en)`: matches Python — first the operator and operand expressions are evaluated, then the function that is the value of the operator is applied to the arguments that are the values of the operands. Scheme exclusively uses prefix notation, nested and multi-line freely:
  ```scheme
  (+ (* 3 5) (- 10 6))
  (+ (* 3 (+ (* 2 4) (+ 3 5))) (+ (- 10 7) 6))
  ```
- **`(if <predicate> <consequent> <alternative>)`**: evaluate `<predicate>` first. If it evaluates to a true value, evaluate `<consequent>` and return its value; otherwise evaluate `<alternative>` and return its value. Only one branch is ever evaluated.
- **`(and <e1> ... <en>)`**: evaluate the `<e>`'s one at a time, left to right. If any evaluates to false, the value of the whole expression is false and the remaining `<e>`'s are not evaluated. If all are true, the value is the value of the last one.
- **`(or <e1> ... <en>)`**: evaluate left to right. If any `<e>` evaluates to a true value, that value is returned and the rest are not evaluated. If all evaluate to false, the value is false.
- **`(not <e>)`**: true when `<e>` evaluates to false, and false otherwise.
- **`(define <name> <expr>)`**: bind the symbol to the value of the expression — `(define pi 3.14)` then `(* pi 2)`.
- **`(define (<name> <formal parameters>) <body>)`**: procedure definition. `<name>` is a symbol to be associated with the procedure in the environment; `<formal parameters>` are the names used within the body to refer to the corresponding arguments; `<body>` is an expression that yields the value of the application when the formals are replaced by actual arguments. Name and formals are grouped in parentheses just as in an actual call.
- **`(lambda (<formal-parameters>) <body>)`**: creates a procedure exactly like `define` does, except unnamed. These are equivalent:
  ```scheme
  (define (plus4 x) (+ x 4))
  (define plus4 (lambda (x) (+ x 4)))
  ```
  A lambda expression can itself be the operator: `((lambda (x y z) (+ x y (square z))) 1 2 3)` → `12`.
- **`(begin <e1> ... <en>)`**: allows a single expression to include multiple sub-expressions — used for sequencing (e.g. turtle commands).
- Booleans are `#t` (or `true`) and `#f` (or `false`); comparisons use prefix notation too: `(>= 2 1)` → `#t`.

## Key Concepts
- **Prefix notation**: operator first, always parenthesized; operators are often symbols like `+`, `*`.
- **Special form**: call-like syntax, different evaluation procedure (`if`, `and`, `or`, `not`-adjacent forms, `define`, `lambda`, `begin`).
- **Procedure**: Scheme's term for a function.
- **Short-circuit booleans**: `and` returns the first false (else the last value); `or` returns the first true value.
- **cons / car / cdr / nil**: pair constructor, first element, rest, empty list.
- **null?**: predicate for the empty list; the base case of list recursion.
- **Quoted expression**: any expression that is not evaluated; symbols as data.
- **Symbolic data**: one of Scheme's strengths — working with arbitrary symbols, not just numbers.
- **Lexical scoping**: local `define`s inside a body see enclosing formals, same rules as Python.
- **Turtle graphics**: position-and-heading drawing environment from Logo (another Lisp dialect); `forward`/`fd`, `right`/`rt`, `lt`, `penup`, `pendown`.

## Worked Examples

### 1. Square root by iterative improvement (local definitions + recursion)
```scheme
(define (average x y)
  (/ (+ x y) 2))
(define (abs x)
    (if (< x 0)
        (- x)
        x))
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
(sqrt 9)
```
Walkthrough: `(sqrt 9)` binds `x` to 9 and calls `(sqrt-iter 1.0)`. Each round, `good-enough?` checks whether `(square guess)` is within 0.001 of `x`; if not, `improve` averages the guess with `x/guess` (Newton's method) and `sqrt-iter` recurses on the improved guess: 1.0 → 5.0 → 2.6 → 1.9... → ≈3.0. Iteration is expressed purely as recursion; the inner procedures see `x` lexically.

### 2. Building and dissecting lists
```scheme
(define x (cons 1 2))   ; x => (1 . 2); (car x) => 1; (cdr x) => 2
(cons 1 (cons 2 (cons 3 (cons 4 nil))))  ; => (1 2 3 4)
(list 1 2 3 4)                            ; same list
(define one-through-four (list 1 2 3 4))
(car one-through-four)        ; 1
(cdr one-through-four)        ; (2 3 4)
(car (cdr one-through-four))  ; 2
(cons 10 one-through-four)    ; (10 1 2 3 4)
```
Standard sequence operations fall out of `null?` recursion:
```scheme
(define (length items)
  (if (null? items)
      0
      (+ 1 (length (cdr items)))))
(define (getitem items n)
  (if (= n 0)
      (car items)
      (getitem (cdr items) (- n 1))))
(define squares (list 1 4 9 16 25))
(length squares)    ; 5
(getitem squares 3) ; 16
```
Trace of `(getitem squares 3)`: n=3 → cdr, n=2 → cdr, n=1 → cdr, n=0 → `(car '(16 25))` = 16.

### 3. Quotation — values vs symbols
```scheme
(define a 1)
(define b 2)
(list a b)    ; (1 2)   — values
(list 'a 'b)  ; (a b)   — symbols
(list 'a b)   ; (a 2)   — mixed
(list 'define 'list)  ; (define list) — even keywords are just symbols when quoted
(car '(a b c))  ; a
(cdr '(a b c))  ; (b c)
```
You can't construct the list `(a b)` with `(list a b)` — that takes the values of `a` and `b`. Quotation mirrors the philosophical use/mention distinction: a dog runs and barks; the word "dog" is a linguistic construct designating such things. Quotation lets Scheme talk about Scheme — and lets you type compound objects directly in printed-list notation.

### 4. Sierpinski triangle (mutual recursion with turtle graphics)
```scheme
(define (repeat k fn)
  (if (> k 0)
      (begin (fn) (repeat (- k 1) fn))
      nil))
(define (tri fn)
  (repeat 3 (lambda () (fn) (lt 120))))
(define (sier d k)
  (tri (lambda ()
         (if (= k 1) (fd d) (leg d k)))))
(define (leg d k)
  (sier (/ d 2) (- k 1))
  (penup)
  (fd d)
  (pendown))
(sier 400 6)
```
`repeat` runs a thunk k times using `begin`; `tri` repeats a drawing thunk 3 times with a 120-degree left turn after each; `sier` draws a plain triangle (`fd d`) at depth 1, else a triangle made of `leg` calls; `leg` recursively calls `sier` at half the length and one less depth to fill the first half of the leg, then lifts the pen, moves to the next vertex, and puts the pen down. The mutual recursion between `sier` and `leg` yields the full fractal — four tiny definitions.

## Mental Models
- Scheme is Python with only expressions: the call-expression evaluation rule is identical (evaluate operator and operands, apply).
- Quotation is the use/mention distinction from philosophy.
- Use nested `define` + recursion where Python would use a loop — iteration is recursion here (`sqrt-iter`).
- A procedure's name is incidental: `define`-with-name is sugar for binding a `lambda`.

## Common Errors / Anti-patterns
- **Evaluating a special form like a call**: `(if p c a)` must not evaluate both branches; `and`/`or` must short-circuit. An interpreter that evaluates all subexpressions eagerly breaks recursion base cases (`sqrt-iter` would never terminate).
- **Confusing quotation levels**: `(list a b)` builds a list of values; `(list 'a 'b)` builds a list of symbols; `'(a b)` is the whole list quoted. Mixing these up produces data that looks right but contains bound values instead of symbols (or vice versa).
- **Reaching for mutation**: this subset has none — express state change through recursion and new values. (Full Scheme has mutation, vectors, and maps; the text's subset deliberately omits them.)
- **Forgetting `nil` terminates a list**: `(cons 1 2)` is a pair, not a one-element list; proper lists end in `nil`/`'()`.

## Key Takeaways
1. Classify every form first: call expression or special form — the evaluation rule differs, and the special-form rules above are quoted verbatim-faithfully because 3.5 implements them.
2. Lists are nested pairs ending in nil; write list code as recursion with a `null?` base case (`length`, `getitem`).
3. Quotation turns code and symbols into data — the prerequisite for interpreters that read programs as lists.
4. Procedures are first-class: `lambda` anywhere a value goes, including operator position.
5. This immutable expression-only subset is enough to express most ideas from the whole text.

## Connects To
- **Section 3.4**: Calculator borrows Scheme's call-expression syntax; the interpreter represents these programs as `Pair` lists — exactly the cons-pairs introduced here.
- **Section 3.5**: the special-form rules (`define`, `lambda`, `if`, `and`, `or`) get implemented in `scheme_eval`; quotation becomes the bridge from programs to data.
- **Section 3.1**: Scheme is the minimal powerful language promised there.
- **Section 1.6/1.7**: higher-order functions and recursion, now in Scheme clothing.
