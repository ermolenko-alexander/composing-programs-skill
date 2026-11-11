# Section 3.4: Interpreters for Languages with Combination

## Core Idea
Metalinguistic abstraction — establishing new languages — is realized by building an interpreter: here a complete Python interpreter for Calculator, a Scheme-syntax arithmetic language, structured as tokenizer → parser → evaluator → REPL.

## Frameworks Introduced
- **Metalinguistic abstraction**: establishing new languages and implementing them by constructing interpreters; an interpreter is "a function that, when applied to an expression of the language, performs the actions required to evaluate that expression".
  - When to use: when a problem is better expressed in its own notation than in the host language.
  - How: define syntax, represent expressions as data, write eval/apply.
- **Two-stage parsing (lexical + syntactic analysis)**:
  - When to use: turning raw text into expression trees.
  - How: the **lexical analyzer** (tokenizer, `tokenize_line`) partitions a string into tokens line-by-line, iteratively; the **syntactic analyzer** (`scheme_read`) consumes the token sequence tree-recursively, possibly spanning multiple lines, producing nested `Pair` expression trees.
- **Dispatching on type / self-evaluating expressions**: `calc_eval` inspects the form of the expression — numbers are self-evaluating and returned directly; `Pair`s are call expressions evaluated by mapping `calc_eval` over operands then calling `calc_apply`.
- **Read-eval-print loop (REPL)**: read an expression, evaluate it, print the result, loop; "parameterized by a parsing function, an evaluation function, and the exception types handled" — the same structure serves every language.

## Key Concepts
- **Expression tree as data**: for the first time expressions are explicit data (nested `Pair` instances), not just conceptual entities.
- **Pair / nil**: Python class representing Scheme pairs; single `nil` instance for the empty list; `repr` is a Python expression, `str` is a Scheme expression; supports `len`, indexing, and `.map`.
- **Token**: minimal syntactic unit (symbol, numeral, delimiter); Scheme tokens are delimited by whitespace, parentheses, dots, quotes.
- **scheme_read / read_tail**: mutually recursive parser functions — `scheme_read` handles base cases (EOFError on empty input, primitives) and recurses into `read_tail` on `(`; `read_tail` reads elements until `)`.
- **Buffer**: object collecting tokens across lines so syntactic analysis can span multiple lines.
- **calc_eval / calc_apply**: eval evaluates numbers and call expressions; apply dispatches on the operator symbol and reduces the argument list.
- **Self-evaluating**: primitive expressions needing no further evaluation step (numbers here; strings, booleans in general languages).

## Mental Models
- Think of the interpreter as a pipeline: text → tokens (iterative, per line) → expression tree (tree-recursive, multi-line) → value (tree-recursive eval).
- Think of eval's job as "make proper calls to apply": evaluate operand sub-expressions first, pass values — apply must receive values, never expressions.
- Lexical analysis is flat and local; syntactic analysis is hierarchical and global — keep them separate components.
- Informative error messages are a feature: raising typed exceptions with descriptions "improves the usability of an interpreter substantially".

## Anti-patterns
- **Passing operand expressions to apply**: `calc_apply` needs a list of values; evaluation of operands is eval's job.
- **A REPL that dies on bad input**: catch the language's exceptions, print, and restart the loop so users can revise their expressions.
- **Conflating tokenizing with parsing**: multi-line expressions and nesting require the tree-recursive reader, not the tokenizer.

## Code Examples
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
- **What it demonstrates**: dispatch on expression form; recursive operand evaluation feeding apply.

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
- **What it demonstrates**: a robust REPL — errors reported without exiting; interrupt/EOF terminate cleanly.

## Worked Example
The full Calculator pipeline for `(+ 1 (* 2.3 45))`:
1. `tokenize_line('(+ 1 (* 2.3 45))')` → `['(', '+', 1, '(', '*', 2.3, 45, ')', ')']` — numerals already converted to numeric types.
2. `scheme_read(Buffer(tokenize_lines(lines)))` → `Pair('+', Pair(1, Pair(Pair('*', Pair(2.3, Pair(45, nil))), nil)))`, printing as `(+ 1 (* 2.3 45))` — works even when the expression spans two lines.
3. `calc_eval` maps itself over operands, then `calc_apply('+', args)` reduces: `+` folds `add` from 0, `*` folds `mul` from 1 (so `(+)` is 0, `(*)` is 1); `-` and `/` require ≥1 argument — negate/invert with one, fold `sub`/`truediv` from the first otherwise.
4. The REPL prints the value; malformed input yields informative errors, e.g. `)` → `SyntaxError: unexpected token: )`, `2.3.4` → `ValueError: invalid numeral: 2.3.4`, `(/ 1 0)` → `ZeroDivisionError: division by zero`.

## Key Takeaways
1. To interpret, first represent programs as data — nested Scheme lists become nested `Pair` instances.
2. Split parsing into an iterative tokenizer and a tree-recursive reader; recursion in the reader generates the hierarchy the evaluator consumes.
3. Structure evaluation as eval (dispatch on expression form, evaluate operands) plus apply (dispatch on operator, combine values).
4. Variadic operators fall out naturally from reduce with the right identity (0 for +, 1 for *).
5. Wrap the REPL body in try/except so errors inform rather than kill; the REPL skeleton is language-independent.

## Connects To
- **Section 3.2**: Calculator reuses Scheme's call-expression syntax and prefix operators.
- **Section 3.3**: the REPL's robustness is built entirely from the exception mechanism.
- **Section 3.5**: the same reader and REPL generalize; eval grows special forms and environments, apply grows user-defined procedures.
