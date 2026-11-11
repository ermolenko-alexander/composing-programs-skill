# Section 3.5: Interpreters for Languages with Abstraction

## Core Idea
To go from Calculator (combination only) to a general language, the interpreter must support abstraction — binding names to values and defining new operations. That means adding special forms, user-defined procedures, and the environment model of computation to the eval/apply structure. Unlike 3.4's complete source listing, this section is descriptive: the companion project builds the fully functional Scheme interpreter it describes.

## Structure (3.5.1)
A Scheme interpreter shares much of Calculator's structure: a parser produces an expression; an evaluator inspects its form; call expressions route to a function that applies a procedure to arguments. The differences live in special forms, user-defined functions, and the environment model.

**Parsing.** The `scheme_reader` and `scheme_tokens` modules from Calculator are nearly sufficient, but must be extended to parse quotation and dotted lists:
```python
>>> read_line("(car '(1 . 2))")
Pair('car', Pair(Pair('quote', Pair(Pair(1, 2), nil)), nil))
```
Note what the quote mark becomes: a two-element list whose first element is the symbol `quote` — syntax desugared into ordinary list structure.

## Evaluation Rules (the author's procedures)

**scheme_eval** evaluates an expression `expr` in the current environment `env`, one expression at a time. The form of a combination is determined by inspecting its first element; each special form has its own evaluation rule. Simplified implementation (error checking and most special forms removed; the complete one is in the companion project):
```python
def scheme_eval(expr, env):
    """Evaluate Scheme expression expr in environment env."""
    if scheme_symbolp(expr):
        return env[expr]
    elif scheme_atomp(expr):
        return expr
    first, rest = expr.first, expr.second
    if first == "lambda":
        return do_lambda_form(rest, env)
    elif first == "define":
        do_define_form(rest, env)
        return None
    else:
        procedure = scheme_eval(first, env)
        args = rest.map(lambda operand: scheme_eval(operand, env))
        return scheme_apply(procedure, args, env)
```
Dispatch order: symbol → environment lookup; other atom → self-evaluating; special-form keyword in first position → that form's handler (`do_lambda_form` returns the procedure; `do_define_form` binds and the form returns `None`); otherwise → call expression: evaluate the operator, map `scheme_eval` over operands (same `env`), then apply.

**scheme_apply** (procedure application) is considerably more general than `calc_apply`. It applies two kinds of procedures:
- **PrimitiveProcedure** — implemented in Python; its instance attribute `fn` is bound to a Python function called whenever the procedure is applied; it may or may not require access to the current environment.
- **LambdaProcedure** — implemented in Scheme; its `body` attribute is a Scheme expression evaluated whenever the procedure is applied. To apply it to arguments: add a **new frame** to the environment in which the formal parameters are bound to the arguments, then evaluate the body with `scheme_eval` in that new environment.

**Eval/apply recursion.** `scheme_eval` and `scheme_apply` are mutually recursive: evaluation requires application whenever a call expression is encountered; application uses evaluation to turn operand expressions into arguments and to evaluate the bodies of user-defined procedures. The cycle ends at language primitives — eval's base case is a primitive expression (and some special forms add non-recursive base cases); apply's base case is a primitive procedure. This structure between an eval that processes expression forms and an apply that processes functions and their arguments constitutes the essence of the evaluation process.

## Environments (3.5.2)
The `Frame` class forms environments. Each `Frame` instance represents an environment in which symbols are bound to values:
- a dictionary of `bindings`;
- a `parent` frame, `None` for the global frame.

Bindings are never accessed directly, only through two methods:
- **lookup**: Chapter 1's look-up procedure — match the symbol against the current frame's `bindings`; if found, return the bound value; if not, proceed to the `parent` frame (and so on up the chain).
- **define**: always binds a symbol to a value **in the current frame**, never a parent.

(Implementing `lookup` and using `define` are the project's exercises.)

## Key Concepts
- **Special form dispatch**: inspect a combination's first element (`lambda`, `define`, `quote`, `if`, ...); each has its own rule, applied before any operand evaluation.
- **Symbol evaluation**: `env[expr]` — environment lookup via the frame chain.
- **Self-evaluating atom**: non-symbol atoms return themselves.
- **do_define_form / do_lambda_form**: Python handlers for the special forms; `define` returns None.
- **PrimitiveProcedure vs LambdaProcedure**: Python-implemented (`fn` attribute) vs Scheme-implemented (`body` attribute + formals + parent environment).
- **Frame / lookup / define**: environment node; chain-walking read; current-frame-only write.
- **Quotation and dotted lists**: parser extensions beyond Calculator; `'x` reads as `(quote x)`.
- **Universal machine**: the interpreter emulates any machine described to it as a program.
- **run / eval / exec**: explicit bridges from data back to program (Scheme's `run`; Python's `eval` for expressions, `exec` for statements).

## Worked Examples

### 1. Defining factorial, then calling it through the Frames
```scheme
(define (factorial n)
  (if (= n 0) 1 (* n (factorial (- n 1)))))
(factorial 5)
```
**The define form**, evaluated by `do_define_form`, takes several steps:
1. Check the format: a well-formed Scheme list with at least two elements after the keyword `define`.
2. Analyze the first element — here a `Pair` — to find the function name `factorial` and formal parameter list `(n)`.
3. Create a `LambdaProcedure` with the supplied formal parameters, body, and parent environment (here, the environment is only the global frame).
4. Bind the symbol `factorial` to this procedure in the **first frame of the current environment** (the global frame).

**The call** `(factorial 5)`: `scheme_eval` sees a call expression; evaluating the operator symbol `factorial` looks it up — the `LambdaProcedure` just created; the args evaluate to the one-element Scheme list `(5)`. `scheme_apply` then creates a new frame **extending the global frame** (the parent environment of the `factorial` procedure), binds `n` to 5 in it, and evaluates the body in that environment. The body's `if` tests `(= n 0)`; false, so `(* n (factorial (- n 1)))` evaluates — calling `factorial` again, each recursive application making another frame with its own `n`, until `n = 0` returns 1 and the multiplications unwind to 120.

### 2. Evaluating a lambda definition and call
`(define plus4 (lambda (x) (+ x 4)))` → eval sees `define`, whose handler evaluates the lambda form: `do_lambda_form` packages formals `(x)`, body `(+ x 4)`, and the current environment into a `LambdaProcedure`, which `define` binds to `plus4` in the current frame. `(plus4 3)` → lookup yields the LambdaProcedure; a new frame with `x → 3` extends the procedure's parent environment; evaluating `(+ x 4)` there looks up `x` (current frame) and `+` (found up the chain in the global frame, a PrimitiveProcedure), and applying `+` calls its Python `fn` — the base case that terminates the recursion.

### 3. Data as programs (3.5.3)
A program can be viewed as the description of an abstract machine. The factorial definition describes a machine containing parts that decrement, multiply, and test for equality, together with a two-position switch and another factorial machine inside it (the machine is infinite because it contains itself). The equivalent Python:
```python
def factorial(n):
    return 1 if n == 1 else n * factorial(n - 1)
```
The Scheme interpreter is then a very special machine that takes as input a description of a machine and configures itself to emulate it — a **universal machine**, bridging data objects and the language itself. From the user's perspective `(+ 2 2)` is an expression to evaluate; from the interpreter's perspective it is simply data manipulated according to well-defined rules. The distinction can be deliberately ignored: Scheme's `run`, Python's `eval`/`exec`:
```python
>>> eval('2+2')
4
>>> 2+2
4
```
Constructing and evaluating expressions during execution is common in Scheme and a valuable tool in any dynamic language.

## Mental Models
- A program = a machine description; the interpreter = the machine that emulates any described machine.
- Each procedure call = "new frame extending the **procedure's parent environment**" — never the caller's environment. That single choice is what makes scope lexical.
- The first element of a combination is the dispatch key: a special-form keyword, or an expression evaluating to a procedure.
- Hold both views of `(+ 2 2)` at once — program to the user, data to the interpreter; that duality is the point, not a confusion.
- `lookup` walks up; `define` stays put. Reads are chain-wide, writes are local.

## Common Errors / Anti-patterns
- **Evaluating special forms as calls**: `define`, `lambda`, `quote`, `if` must be intercepted before the call-expression path; their operands are not all evaluated (eagerly evaluating `if` branches breaks recursion; evaluating a `quote`'s operand defeats quotation).
- **Forgetting the environment parameter**: every recursive `scheme_eval` call needs the right `env` — operands use the current env, a LambdaProcedure body uses the new frame over the procedure's parent env.
- **Binding in the wrong frame**: `define` binds in the current frame only; it is `lookup` that walks the parent chain.
- **Extending the caller's environment when applying a LambdaProcedure**: gives dynamic scope instead of lexical.
- **Confusing quotation levels in the reader**: `'(1 . 2)` must become `(quote (1 . 2))` as list structure — a `Pair('quote', Pair(Pair(1, 2), nil))` — not a pre-evaluated value.

## Key Takeaways
1. Abstraction = names + user-defined procedures; both ride on the environment model (Frame with lookup/define).
2. Eval dispatches on expression form, apply dispatches on procedure kind; primitive expressions and primitive procedures terminate the mutual recursion.
3. Applying a LambdaProcedure = new frame over the procedure's parent environment, formals bound to arguments, body evaluated there via `scheme_eval`.
4. `define` of a function = analyze name+formals, build a LambdaProcedure capturing the defining environment, bind in the current frame.
5. The interpreter is a universal machine; programs are its data, and `run`/`eval`/`exec` let programmers exploit that deliberately.
6. The jump from Calculator is mostly special forms, environments, and LambdaProcedure — parser and REPL barely change (add quotation and dotted lists).

## Connects To
- **Section 3.4**: shares the parser (`scheme_reader`, `scheme_tokens`) and the eval/apply split; `scheme_apply` generalizes `calc_apply` from operator symbols to procedure objects.
- **Section 3.2**: implements the Scheme special-form rules (`define`, `lambda`, `if`) stated there; quotation becomes a parsing requirement.
- **Section 3.1**: delivers the promised insight — the interpreter determining Scheme's meaning is just another (Python) program, built as two mutually recursive functions.
- **Chapter 1 (1.3, 1.6)**: the environment model of computation being implemented here is exactly the one introduced for Python.
