# Section 3.4: Interpreters for Languages with Combination

## Core Idea
Metalinguistic abstraction — establishing new languages — is realized by building an interpreter: a function that, when applied to an expression of the language, performs the actions required to evaluate it. Here: a complete Python interpreter for Calculator, a Scheme-syntax arithmetic language, structured as tokenizer → parser → evaluator → REPL. Calculator is a limited Scheme subset; 3.5 then sketches full Scheme, implementing the same environment model of evaluation introduced for Python in Chapter 1.

## The Calculator Language
Calculator covers addition, subtraction, multiplication, and division with Scheme's call-expression syntax and operator behavior:
- `+` and `*` take an arbitrary number of arguments: `(+ 1 2 3 4)` → 10, `(+)` → 0, `(*)` → 1, `(* 1 2 3 4)` → 24.
- `-` has two behaviors: one argument → negation (`(- 3)` → −3); two or more → subtract all but the first from the first (`(- 10 1 2 3)` → 4).
- `/` mirrors it: one argument → multiplicative inverse (`(/ 10)` → 0.1); more → divide all but the first into the first (`(/ 30 5 2)` → 3).
- Nesting works by the standard rule: `(- 100 (* 7 (+ 8 (/ -12 -3))))` → 16.0.

The interpreter is a Python program taking string lines as input, returning the result of evaluating them, and raising an appropriate exception for malformed expressions.

## Frameworks Introduced
- **Expression trees as data (3.4.2)**: until now expression trees were conceptual entities; to write an interpreter we must operate on expressions as data. Primitive Calculator expressions are numbers (`int`/`float`) or operator-symbol strings; all combined expressions are call expressions — Scheme lists with the operator first, operands after.
- **Two-stage parsing (3.4.3)**: a parser = lexical analyzer + syntactic analyzer. The tokenizer's output sequence is consumed by the reader.
- **Dispatching on type (3.4.4)**: `calc_eval` dispatches on the form of the expression; `calc_apply` dispatches on the operator symbol.
- **Read-eval-print loop**: a largely interpreter-independent interaction mode — parameterized by a parsing function, an evaluation function, and the exception types handled; beyond those, all REPLs share the same structure.

## Key Concepts
- **Pair / nil**: Python class representing Scheme pairs (like the earlier `Rlist`), defined in `scheme_reader`; a single `nil` instance represents the empty list. `repr` strings are Python expressions, `str` strings are Scheme expressions:
  ```python
  >>> s = Pair(1, Pair(2, nil))
  >>> s
  Pair(1, Pair(2, nil))
  >>> print(s)
  (1 2)
  >>> len(s); s[1]; print(s.map(lambda x: x+4))
  2
  2
  (5 6)
  ```
  Pairs nest, so they suffice for all Scheme expressions (nested lists):
  ```python
  >>> expr = Pair('+', Pair(Pair('*', Pair(3, Pair(4, nil))), Pair(5, nil)))
  >>> print(expr)                       # (+ (* 3 4) 5)
  >>> print(expr.second.first)          # (* 3 4)
  >>> expr.second.first.second.first    # 3
  ```
- **Token**: minimal syntactic unit (names, symbols, numerals, delimiters). Scheme tokens are delimited by whitespace, parentheses, dots, or single quotation marks; delimiters are themselves tokens.
- **tokenize_line** (in `scheme_tokens`): analyzes a line character by character, validating symbols and numerals, converting multi-character numerals to numeric types. Lexical analysis is iterative and per-line.
- **scheme_read / read_tail** (in `scheme_reader`): the tree-recursive syntactic analyzer; must consider an entire expression that may span multiple lines.
- **Buffer** (in `buffer`): collects tokens across lines into one object the reader can consume.
- **calc_eval / calc_apply** (in `scalc`): the eval/apply pair; helpers `simplify`, `reduce`, `as_scheme_list` defined in the companion code.
- **Self-evaluating**: primitive expressions requiring no further evaluation step — numbers in Calculator; a general language also has strings, booleans, etc.

## Evaluation Rules (the author's procedures)

**Parsing.** `scheme_read` first checks base cases: empty input raises `EOFError`; primitive expressions are returned directly. On a `(` token it calls `read_tail`. `read_tail` reads from the same `src`, expecting a list already begun; its base cases are empty input (an error) or a closing parenthesis terminating the list; its recursive case reads the first element with `scheme_read`, the rest with `read_tail`, and returns a `Pair`. This reads all well-formed Scheme lists — enough for Calculator (dotted lists and quoted forms are an exercise). `SyntaxError`s include a description of the problem; informative syntax errors improve usability substantially.

**Evaluation.**
```python
def calc_eval(exp):
    """Evaluate a Calculator expression."""
    if type(exp) in (int, float):
        return simplify(exp)
    elif isinstance(exp, Pair):
        arguments = exp.second.map(calc_eval)
        return simplify(calc_apply(exp.first, arguments))
    else:
        raise TypeError(exp + ' is not a number or call expression')
```
Only two legal syntactic forms: numbers (self-evaluating, returned directly) and call expressions (`Pair`s). A call is evaluated by recursively mapping `calc_eval` over the operand list to get arguments, then applying the operator in `calc_apply`.

**Application.** Each conditional clause applies one operator; wrong argument counts raise `TypeError`:
```python
def calc_apply(operator, args):
    """Apply the named operator to a list of args."""
    if not isinstance(operator, str):
        raise TypeError(str(operator) + ' is not a symbol')
    if operator == '+':
        return reduce(add, args, 0)
    elif operator == '-':
        if len(args) == 0:
            raise TypeError(operator + ' requires at least 1 argument')
        elif len(args) == 1:
            return -args.first
        else:
            return reduce(sub, args.second, args.first)
    elif operator == '*':
        return reduce(mul, args, 1)
    elif operator == '/':
        if len(args) == 0:
            raise TypeError(operator + ' requires at least 1 argument')
        elif len(args) == 1:
            return 1/args.first
        else:
            return reduce(truediv, args.second, args.first)
    else:
        raise TypeError(operator + ' is an unknown operator')
```
`calc_apply` must be passed a list of **values**, never operand expressions — evaluating operands is `calc_eval`'s job:
```python
>>> calc_apply('+', as_scheme_list(1, 2, 3))   # 6
>>> calc_apply('-', as_scheme_list(10, 1, 2, 3))  # 4
>>> calc_apply('*', nil)                        # 1
>>> calc_apply('/', as_scheme_list(40, 5))      # 8.0
```

**REPL.** The bare loop, then the robust one:
```python
def read_eval_print_loop():
    """Run a read-eval-print loop for calculator."""
    while True:
        src = buffer_input()
        while src.more_on_line:
            expression = scheme_read(src)
            print(calc_eval(expression))
```
```python
def read_eval_print_loop():
    """Run a read-eval-print loop for calculator."""
    while True:
        try:
            src = buffer_input()
            while src.more_on_line:
                expression = scheme_read(src)
                print(calc_eval(expression))
        except (SyntaxError, TypeError, ValueError, ZeroDivisionError) as err:
            print(type(err).__name__ + ':', err)
        except (KeyboardInterrupt, EOFError):  # <Control>-D, etc.
            print('Calculation completed.')
            return
```
The first except clause handles `SyntaxError`/`ValueError` raised by `scheme_read` and `TypeError`/`ZeroDivisionError` raised by `calc_eval` — reporting without exiting, so users can revise their expressions (importing `readline` even enables input recall). The second lets Control-C / Control-D terminate cleanly.

## Worked Examples

### 1. Full pipeline for `(+ 1 (* 2.3 45))`
1. **Tokenize**: `tokenize_line('(+ 1 (* 2.3 45))')` → `['(', '+', 1, '(', '*', 2.3, 45, ')', ')']` — numerals already numeric.
2. **Read** (works across lines):
   ```python
   >>> lines = ['(+ 1', '   (* 2.3 45))']
   >>> expression = scheme_read(Buffer(tokenize_lines(lines)))
   >>> expression
   Pair('+', Pair(1, Pair(Pair('*', Pair(2.3, Pair(45, nil))), nil)))
   >>> print(expression)
   (+ 1 (* 2.3 45))
   ```
   Trace: `scheme_read` sees `(`, calls `read_tail`; `read_tail` reads `'+'` (via `scheme_read`), then `1`, then hits `(` so the nested `scheme_read`→`read_tail` builds `(* 2.3 45)`, then `)` closes the outer list.
3. **Evaluate**: `calc_eval` maps itself over `(1 (* 2.3 45))` → the inner call evaluates to `calc_apply('*', (2.3 45))` = 103.5 (reduce `mul` from 2.3); then `calc_apply('+', (1 103.5))` = 104.5.

### 2. `calc_eval(exp)` for `(+ (* 3 4) 5)`
Operands map to `(12 5)` — the nested `Pair('*', ...)` recursively becomes `calc_apply('*', (3 4))` = 12 — then `calc_apply('+', (12 5))` → 17.

### 3. Error-reporting session
```text
> )
SyntaxError: unexpected token: )
> 2.3.4
ValueError: invalid numeral: 2.3.4
> +
TypeError: + is not a number or call expression
> (/ 1 0)
ZeroDivisionError: division by zero
```
Each layer raises its own typed exception (reader: Syntax/Value; evaluator: Type/ZeroDivision); the REPL catches and names them.

## Mental Models
- The interpreter is a pipeline: text → tokens (iterative, per line) → expression tree (tree-recursive, multi-line) → value (tree-recursive eval).
- Eval's job is "make proper calls to apply": compute operand values first; apply never sees expressions.
- Lexical analysis is flat and local; syntactic analysis is hierarchical and global — keep them separate components.
- Recursion in the reader generates the hierarchical structures the evaluator consumes.

## Common Errors / Anti-patterns
- **Passing operand expressions to apply**: `calc_apply` needs values; `calc_apply('+', scheme-list-of-expressions)` silently misbehaves or crashes.
- **A REPL that dies on bad input**: catch the language's exceptions, print, restart the loop.
- **Conflating tokenizing with parsing**: multi-line and nested expressions need the tree-recursive reader; the tokenizer alone can't build structure.
- **Forgetting bare operators are errors**: `+` alone at top level is neither a number nor a call expression — `calc_eval`'s else branch must reject it.
- **Skipping arity checks**: `-` and `/` require at least 1 argument; raising a descriptive `TypeError` beats a confusing crash later.

## Key Takeaways
1. To interpret, first represent programs as data — nested Scheme lists become nested `Pair` instances with dual `repr`/`str` identities.
2. Split parsing into an iterative tokenizer and a tree-recursive reader (`scheme_read`/`read_tail` mutually recursive).
3. Structure evaluation as eval (dispatch on expression form, evaluate operands) plus apply (dispatch on operator, combine values).
4. Variadic operators fall out of reduce with the right identity (0 for `+`, 1 for `*`) and the one-vs-many split for `-` and `/`.
5. Wrap the REPL body in try/except so errors inform rather than kill; the REPL skeleton is language-independent — swap in a new parser, evaluator, and exception list to serve another language.

## Connects To
- **Section 3.2**: Calculator reuses Scheme's call-expression syntax, prefix operators, and cons-pair lists.
- **Section 3.3**: the REPL's robustness is built entirely from the exception mechanism.
- **Section 3.5**: the same reader and REPL generalize; eval grows special forms and environments, apply grows user-defined procedures — closing the eval/apply recursion promised in 3.1.
