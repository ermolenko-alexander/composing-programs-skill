# Section 3.5: Interpreters for Languages with Abstraction

## Core Idea
To go from Calculator (combination only) to a general language, the interpreter must support abstraction — binding names to values and defining new operations — which means adding special forms, user-defined procedures, and the environment model to the eval/apply structure.

## Frameworks Introduced
- **Eval/apply recursion**: `scheme_eval` and `scheme_apply` are mutually recursive — evaluation requires application for call expressions; application uses evaluation for operands and for the bodies of user-defined procedures. "This mutually recursive structure ... constitutes the essence of the evaluation process." The cycle ends at language primitives: primitive expressions (eval's base case) and primitive procedures (apply's base case).
  - When to use: the skeleton of any general-language interpreter.
  - How: eval dispatches on expression form (symbol → env lookup; atom → itself; special form → its rule; otherwise call expression → eval operator and operands, then apply).
- **Environment model via Frame**: each `Frame` has a `bindings` dictionary and a `parent` frame (`None` for global); access only through `lookup` (search current frame, then parent chain) and `define` (always bind in the current frame).
  - When to use: implementing names, scope, and procedure calls.
  - How: applying a `LambdaProcedure` creates a new frame extending the procedure's parent environment, binds formals to arguments there, and evaluates the body in it.
- **Two procedure kinds**: `PrimitiveProcedure` (attribute `fn` bound to a Python function, called on application; may need the environment) vs `LambdaProcedure` (a `body` Scheme expression evaluated in a new frame on application).
- **Data as programs / universal machine**: a program describes an abstract machine; the interpreter is a universal machine that "configures itself to emulate the machine described". The user's programs are the interpreter's data; `run` in Scheme and `eval`/`exec` in Python expose this deliberately.

## Key Concepts
- **Special form dispatch**: the form of a combination is determined by inspecting its first element (`lambda`, `define`, ...); each special form has its own evaluation rule.
- **Symbol evaluation**: a symbol evaluates to `env[expr]` — environment lookup.
- **Self-evaluating atom**: non-symbol atoms return themselves.
- **do_define_form / do_lambda_form**: Python handlers for the `define` and `lambda` special forms; `define` returns None after binding.
- **Frame**: an environment node — bindings dict plus parent pointer.
- **lookup**: Chapter 1's look-up procedure — current frame first, then parent chain.
- **define (method)**: always binds in the current frame, never a parent.
- **Quotation/dotted-list parsing**: the Calculator reader must be extended so `(car '(1 . 2))` parses to `Pair('car', Pair(Pair('quote', Pair(Pair(1, 2), nil)), nil))`.

## Mental Models
- Think of a program as a description of an abstract machine (the factorial definition = a machine of decrement, multiply, equality-test parts plus a nested factorial machine); the interpreter is the machine that emulates any machine so described.
- Think of each procedure call as "new frame extending the procedure's parent environment" — never the caller's environment.
- Use the first element of a combination as the dispatch key: special form keyword or an expression evaluating to a procedure.
- From the user's view `(+ 2 2)` is a program; from the interpreter's view it is data manipulated by well-defined rules — holding both views at once is the point, not a confusion.

## Anti-patterns
- **Evaluating special forms as calls**: `define` and `lambda` must be intercepted before the call-expression path; their operands are not all evaluated.
- **Binding definitions in the wrong frame**: `define` binds in the first (current) frame; lookup, not define, walks the parent chain.
- **Extending the caller's environment instead of the procedure's parent environment** when applying a LambdaProcedure — that would give dynamic scope, not lexical.

## Code Examples
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
- **What it demonstrates**: the full dispatch of a general evaluator — symbols, atoms, special forms, then call expressions feeding scheme_apply.

## Worked Example
`(define (factorial n) (if (= n 0) 1 (* n (factorial (- n 1)))))` then `(factorial 5)`:
1. The define form is handled by `do_define_form`: check the expression is a well-formed list with ≥2 elements after `define`; analyze the first element (a Pair) to get name `factorial` and formals `(n)`; create a `LambdaProcedure` with those formals, the body, and the parent environment (here just the global frame); bind `factorial` to it in the current (global) frame.
2. `(factorial 5)` is a call expression: eval looks up `factorial` (the LambdaProcedure), evaluates args to the Scheme list `(5)`, and `scheme_apply` creates a new frame extending the global frame, binds `n` to 5, and evaluates the body there — recursing through eval/apply until `(= n 0)` ends the cycle.

## Key Takeaways
1. Abstraction = names + user-defined procedures; both ride on the environment model (Frame with lookup/define).
2. Eval dispatches on form, apply dispatches on procedure kind; primitives terminate the mutual recursion.
3. Applying a user-defined procedure = new frame over the procedure's parent environment with formals bound to arguments, then eval the body.
4. The interpreter is a universal machine: feed it a machine description (program) and it emulates that machine.
5. Programs are data — `eval('2+2')` and `2+2` agree; constructing and evaluating expressions at runtime is a powerful, general tool.
6. The jump from Calculator is mostly special forms, environments, and LambdaProcedure — parser and REPL barely change (add quotation and dotted lists).

## Connects To
- **Section 3.4**: shares the parser (`scheme_reader`, `scheme_tokens`) and the eval/apply split; `scheme_apply` generalizes `calc_apply`.
- **Section 3.2**: implements the Scheme special forms (`define`, `lambda`) introduced there; quotation becomes a parsing requirement.
- **Section 3.1**: delivers the promised insight — the interpreter determining Scheme's meaning is just another (Python) program.
