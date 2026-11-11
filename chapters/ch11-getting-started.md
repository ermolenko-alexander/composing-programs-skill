# Section 1.1: Getting Started

## Core Idea
All computing begins with representing information, specifying logic to process it, and designing abstractions that manage the complexity of that logic — and mastering these requires understanding precisely how interpreters carry out computational processes.

## Frameworks Introduced
- **Statements & Expressions**: computer programs consist of instructions that either compute some value or carry out some action. Expressions describe computations and are *evaluated* to a value; statements describe actions and are *executed* by the interpreter.
  - When to use: classify every line of code you read or write into one of these two roles before reasoning about it.
  - How: if it produces a value when entered interactively (Python prints it), it's an expression; if it changes state (binds a name, imports, prints) without a value, it's a statement.
- **Guiding principles of debugging** (the author's four):
  - When to use: any time output is unexpected or an error appears.
  - How: (1) *Test incrementally* — try every small component as soon as you write it; (2) *Isolate errors* — trace the error to the smallest fragment of code before fixing; (3) *Check your assumptions* — interpreters execute instructions to the letter, so unexpected output means a belief about behavior is wrong; verify assumptions explicitly; (4) *Consult others* — shared problem-solving spreads knowledge.
- **Core concept trilogy**: functions, objects, interpreters — the three primary topics of the book's first three chapters, closely related ("functions are objects, objects are functions, and interpreters are instances of both").

## Key Concepts
- **Interpreter**: a program that evaluates compound expressions via a precise, predictable procedure; it is what endows code with meaning.
- **Interactive session**: typing code after the `>>>` prompt; Python evaluates and prints the result on the next line.
- **Import statement**: loads functionality from a module and makes names (e.g. `urlopen`) available.
- **Assignment statement**: associates a name with the value of the expression to the right of `=`.
- **Function**: encapsulates logic that manipulates data; complexity is "tucked away" so a complex process is applied with a simple expression.
- **Object**: seamlessly bundles data with the logic that manipulates it (e.g. a `set`); manages complexity of both.
- **Debugging**: learning to interpret errors and diagnose the cause of unexpected output.

## Mental Models
- Think of a function as a black box whose internal process may be complex but whose application is one simple expression.
- Think of the computer as "powerful + stupid" (Stanford CS101 equation): billions of simple, rigid, mechanical operations per second, no insight — your insight is the program.
- Use incremental experimentation at the interpreter when learning any new construct — trying a function tells you more than its documentation at first.

## Anti-patterns
- **Being intimidated by the machine**: the computer is purely mechanical; errors come from rigid literal execution of your text, not hidden intelligence.
- **Fixing before isolating**: correcting code before tracing the error to the smallest fragment wastes effort and hides the real cause.
- **Writing large blocks before testing**: violates "test incrementally"; well-written programs are small testable modular components.

## Code Examples
```python
>>> from urllib.request import urlopen
>>> shakespeare = urlopen('http://composingprograms.com/shakespeare.txt')
>>> words = set(shakespeare.read().decode().split())
>>> {w for w in words if len(w) == 6 and w[::-1] in words}
{'redder', 'drawer', 'reward', 'diaper', 'repaid'}
```
- **What it demonstrates**: a handful of statements and expressions (import, assignment, chained methods, set comprehension) process the full text of Shakespeare — generality of interpreters and the power of composition.

## Worked Example
The first example finds six-letter Shakespearian words that are also words reversed. Steps: an import statement loads `urlopen`; an assignment binds `shakespeare` to the opened URL of all 37 plays; a second assignment chains `read` → `decode` → `split`, each operating on the previous intermediate result, and collects 33,721 unique words into a `set`; finally a compound expression filters words of length 6 whose reversal (`w[::-1]`) is also in the set, yielding `{'redder', 'drawer', 'reward', 'diaper', 'repaid'}`. Each language feature (statements, functions, objects, compound expressions) appears in miniature.

## Key Takeaways
1. Classify code as expression (evaluated → value) or statement (executed → action) before reasoning about it.
2. Functions are the chapter's primary subject: they hide process complexity behind a simple application.
3. Debug by the four principles: test incrementally, isolate errors, check assumptions, consult others.
4. The interpreter does exactly what you wrote — unexpected behavior means a wrong assumption, not a capricious machine.
5. Experiment interactively; the `>>>` session is the fastest teacher.

## Connects To
- **Section 1.2**: rebuilds the language element by element, formalizing expressions, call expressions, and names previewed here.
- **Section 1.5 (Testing)**: develops "test incrementally" into doctests and assertions.
