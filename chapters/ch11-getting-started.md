# Section 1.1: Getting Started

## Core Idea
All computing begins with representing information, specifying logic to process it, and designing abstractions that manage the complexity of that logic — and mastering these fundamentals requires understanding precisely how computers interpret computer programs and carry out computational processes. The text adapts *Structure and Interpretation of Computer Programs* (SICP, Abelson & Sussman) to Python, introducing Python's features "in step with techniques for abstraction and a rigorous model of computation."

## Frameworks Introduced

### Statements & Expressions (the fundamental classification)
Python code consists of expressions and statements. Broadly, computer programs consist of instructions to either:
1. **Compute some value** — expressions typically describe computations. When Python *evaluates* an expression, it computes the value of that expression.
2. **Carry out some action** — statements typically describe actions. When the Python interpreter *executes* a statement, it carries out the corresponding action.

- When to use: classify every line of code you read or write into one of these two roles before reasoning about it.
- How: if entering it interactively makes Python print a value on the next line, it's an expression; if it changes state (binds a name, imports functionality) without itself having a value, it's a statement.

### The Core Concept Trilogy: Functions, Objects, Interpreters
The book's organizing structure — each is the primary topic of one chapter:
- **Functions** (Chapter 1): functions encapsulate logic that manipulates data. The process by which a web address becomes the text of 37 plays may be complex, "but we can apply that process using only a simple expression because that complexity is tucked away within a function."
- **Objects** (Chapter 2): an object "seamlessly bundles together data and the logic that manipulates that data, in a way that manages the complexity of both." A `set` is an object supporting set operations like intersection and membership.
- **Interpreters** (Chapter 3): evaluating compound expressions requires a precise procedure that interprets code in a predictable way. A program implementing such a procedure is an interpreter. Compared to other programs, interpreters are unique in their *generality* — Python was not designed with Shakespeare in mind, yet processes his complete plays in four lines.
- The punchline: "functions are objects, objects are functions, and interpreters are instances of both." Still, developing a clear understanding of each concept separately is critical to mastering programming.

### The Four Guiding Principles of Debugging
Learning to interpret errors and diagnose the cause of unexpected output is called *debugging*. The author's principles, near-verbatim:
1. **Test incrementally**: every well-written program is composed of small, modular components that can be tested individually. Try out everything you write as soon as possible to identify problems early and gain confidence in your components.
2. **Isolate errors**: an error in the output of a statement can typically be attributed to a particular modular component. When diagnosing a problem, trace the error to the smallest fragment of code you can *before* trying to correct it.
3. **Check your assumptions**: interpreters carry out your instructions to the letter — no more and no less. Output is unexpected when the behavior of some code does not match what the programmer believes (or assumes) that behavior to be. Know your assumptions, then focus debugging effort on verifying that your assumptions actually hold.
4. **Consult others**: you are not alone. If you don't understand an error message, ask a friend, instructor, or search engine. If you've isolated an error but can't correct it, ask someone else to look. A lot of valuable programming knowledge is shared in group problem solving.

Incremental testing, modular design, precise assumptions, and teamwork are themes that persist throughout the text.

## Key Concepts
- **Interpreter**: a program that evaluates compound expressions via a precise, predictable procedure; it is what endows code with meaning. Design and implementation of interpreters is Chapter 3's topic.
- **Interactive session**: you type Python *code* after the *prompt* `>>>`; the interpreter reads and executes what you type. Start with `python3` at a terminal (Mac/Unix/Linux) or the Python 3 application (Windows).
- **Interactive controls**: each session keeps a typed history — `<Control>-P` (previous), `<Control>-N` (next); `<Control>-D` exits and discards history; arrow keys cycle history on some systems.
- **Statement**: describes an action; *executed* by the interpreter (e.g., import, assignment).
- **Expression**: describes a computation; *evaluated* to a value. Entering one interactively prints its value on the following line.
- **Import statement**: loads functionality (e.g., `from urllib.request import urlopen` makes available a function that accesses content at a URL).
- **Assignment statement**: associates a name with the value of the expression that follows `=`.
- **Function**: encapsulates logic that manipulates data; complexity tucked away so a complex process is applied with one simple expression. Functions are the primary topic of Chapter 1.
- **Object**: bundles data with the logic manipulating it (e.g., `set`). Primary topic of Chapter 2.
- **Compound expression**: an expression built of parts (e.g., a set comprehension) whose evaluation requires the interpreter's precise procedure.
- **Debugging**: interpreting errors and diagnosing causes of unexpected output, governed by the four principles above.
- **Python 3**: the text uses the most recent stable Python 3; older versions (e.g., 2.7) will not match the descriptions.

## Mental Models
- **Function as black box**: the internal process may be complex, but application is one simple expression — complexity is "tucked away."
- **computer = powerful + stupid** (Francisco Cai and Nick Parlante, Stanford CS101): computers perform billions of operations per second, each extremely rigid, simple, mechanical. The computer "lacks anything like real insight ... it's nothing like the HAL 9000 from the movies." Programming is a person using their *real insight* to build something useful out of these teeny, simple operations.
- **Language as community**: "A language isn't something you learn so much as something you join" (Arika Okrent). Learning Python means joining a million-person developer community that shares conventions facilitating reading, understanding, and extending programs.
- **Interactive experimentation**: the fastest way to learn any construct is to try it at the `>>>` prompt; the interpreter is waiting for your command.

## Anti-patterns
- **Being intimidated by the machine**: the computer is purely mechanical underneath; errors come from rigid, literal execution of your text, not hidden intelligence. Even the smallest spelling and formatting changes cause unexpected output and errors.
- **Fixing before isolating**: correcting code before tracing the error to the smallest fragment wastes effort and hides the real cause (violates principle 2).
- **Writing large blocks before testing**: violates "test incrementally"; well-written programs are small, individually testable modular components.
- **Debugging without naming your assumptions**: if you can't state what you believe the code does, you can't find where belief and behavior diverge.
- **Using Python 2 with this material**: the text's descriptions assume Python 3.

## Code Examples
The complete first example — four lines that exercise statements, functions, objects, and interpreters:
```python
>>> from urllib.request import urlopen
>>> shakespeare = urlopen('http://composingprograms.com/shakespeare.txt')
>>> words = set(shakespeare.read().decode().split())
>>> {w for w in words if len(w) == 6 and w[::-1] in words}
{'redder', 'drawer', 'reward', 'diaper', 'repaid'}
```
What each line demonstrates:
- Line 1 — an **import statement** loading Internet-access functionality, making the function `urlopen` available.
- Line 2 — an **assignment statement** associating `shakespeare` with the value of the expression right of `=`: `urlopen` applied to a URL containing the complete text of Shakespeare's 37 plays in one document.
- Line 3 — another assignment, binding `words` to the set of all 33,721 unique words. The chained commands each operate on an intermediate computational entity: `read` the data from the opened URL, `decode` the data into text, `split` the text into words; all placed in a `set`.
- Line 4 — a **compound expression** (set comprehension) evaluating to all Shakespearian words that are simultaneously a word spelled in reverse. `w[::-1]` enumerates each letter in a word, with `-1` dictating to step backwards.

And the canonical smallest interaction:
```python
>>> 2 + 2
4
```

## Worked Example: Reversible Shakespeare Words, End to End
Goal: find all six-letter words in Shakespeare that are also words when reversed.
1. **Load capability**: `from urllib.request import urlopen` — a statement; executing it carries out the action of making `urlopen` available. Nothing is printed: statements have no value.
2. **Fetch**: `shakespeare = urlopen('http://...')` — the expression right of `=` applies the function to data (a web address), producing another piece of data (an open connection to the full text). The assignment binds the name.
3. **Build the vocabulary**: `words = set(shakespeare.read().decode().split())` — three chained function applications, each consuming the previous intermediate result, then collected into a `set` object. 33,721 unique words.
4. **Query**: the set comprehension filters to words of length 6 whose reversal is also in `words`. Entered interactively, Python evaluates it and prints `{'redder', 'drawer', 'reward', 'diaper', 'repaid'}`.

The lesson in miniature: a handful of statements and expressions, composed, process a large corpus — because functions hide process complexity, objects bundle data with operations, and the interpreter evaluates any compound expression by one general procedure. Python's generality (not designed for Shakespeare) is the hallmark of interpreters.

## Common Errors (what the author explicitly warns about)
- **Rigidity errors**: "even the smallest spelling and formatting changes will cause unexpected output and errors" — expect them when experimenting before you know the full vocabulary and structure of the language.
- **Assumption mismatch**: unexpected output always means some belief about code behavior is wrong — the interpreter did exactly what was written, no more and no less.
- **Version mismatch**: machines often ship with Python 2.7 preinstalled; its behavior will not match this text — install Python 3.

## Key Takeaways
1. Classify code as expression (evaluated → value) or statement (executed → action) before reasoning about it.
2. Functions, objects, and interpreters are the three organizing concepts; functions are this chapter's subject — they hide process complexity behind a simple application.
3. Debug by the four principles: test incrementally, isolate errors, check assumptions, consult others.
4. The interpreter does exactly what you wrote; the computer is "powerful + stupid" — your insight is the program.
5. Experiment interactively; the `>>>` session is the fastest teacher, and history controls (`C-p`/`C-n`) make iteration cheap.

## Connects To
- **Section 1.2 (Elements of Programming)**: rebuilds the language piece by piece, formalizing expressions, call expressions, names, and the evaluation procedure previewed here.
- **Section 1.5.6 (Testing)**: develops "test incrementally" into assertions and doctests.
- **Chapter 2**: objects (like `set`) as the means for bundling data with logic.
- **Chapter 3**: interpreters — implementing the precise evaluation procedure yourself.
