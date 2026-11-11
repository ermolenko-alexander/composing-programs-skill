# Section 2.4: Mutable Data

## Core Idea
Adding state to data lets a single data object evolve independently of the rest of the program, with behavior influenced by its history — the central ingredient of object-oriented programming. Modularity comes from objects that manage their own internal state.

## Frameworks Introduced
- **The object metaphor**: objects combine data values with behavior; they are both information and processes, bundled to represent the properties, interactions, and behaviors of complex things. All values in Python are objects.
- **Mutable vs immutable**: numbers, strings, tuples are immutable; lists and dicts are mutable. Mutation operations change a value in place; they do not create new objects.
- **Identity vs equality**: `is` tests whether two expressions evaluate to the identical object (any change to one reflected in the other); `==` tests equality of contents. Identity is a stronger condition than equality.
- **nonlocal statement**: lets a function *change* (not just access) a binding in an enclosing frame — the first frame in which the name is already bound, excluding local and global.
  - When to use: maintaining state local to a function that evolves over successive calls (`make_withdraw`).
  - How: declare `nonlocal name` before assigning; without it, assignment creates a local binding and triggers `UnboundLocalError` if the name is also read (Python pre-computes each name's frame — all occurrences of a name in a function body must refer to one frame).
- **Referential transparency**: an expression of only pure function calls keeps its value when a subexpression is replaced by its value. Re-binding operations violate it — they change the environment, raising the "sameness and change" problem: a mutable object has an identity distinct from its parts.
- **Message passing**: encapsulate all operations on a data value in one **dispatch function** that takes a message (string naming the behavior) as first argument plus parameters — effectively many functions in one, with local state.
- **Dispatch dictionaries**: replace the conditional dispatch with a dictionary mapping message strings to functions and data; storing state in the dictionary itself avoids `nonlocal`.
- **Constraint propagation / declarative programming**: declare relations among quantities (a network of constraints joined by connectors) instead of one-directional computations; the same network computes any quantity from the others.

## Key Concepts
- **Attribute**: named value that is part of an object, accessed by dot notation `<expression>.<name>`; attribute names are particular to the instance, not the environment.
- **Method**: function-valued attribute; computes its result from its arguments *and* its object (`tues.strftime('%A')`).
- **List mutation methods**: `pop`, `remove`, `append`, `extend`, `insert`, element assignment `s[2] = x`, slice assignment `s[0:2] = [...]`.
- **Sharing**: two names bound to the same list co-refer; mutation through one name affects the other. `list(s)` copies (shallow — shared structure still aliases).
- **Tuple**: immutable sequence; literal `(10,)` for one element, `()` for none; can still contain mutable elements that change; usable as dict keys (lists are not); implicit in multiple assignment.
- **Dictionary**: key-value store; one value per key; keys cannot be or contain mutable values; `get(key, default)`; `keys`/`values`/`items` return iterables; dict comprehension `{x: x*x for x in range(3,6)}`.
- **Local state**: a changing value with particular contents at each point in execution; functions get it via nonlocal bindings in their parent frame.
- **Connector**: object that "holds" a value and participates in constraints; responds to `set_val`, `has_val`, `val`, `forget`, `connect`.
- **Informant**: the source a connector remembers as having set its value; only that source may retract it.

## Mental Models
- To analyze code with non-local assignment, remember: only function calls introduce new frames; assignment statements always change bindings in existing frames. One call to `make_withdraw` = one `balance` binding, shared by all calls to its withdraw.
- Think of each closure from a maker function as an independent object: `wd = make_withdraw(20)` and `wd2 = make_withdraw(7)` have separate parents and separate state; `wd2 = wd` is one object with two names.
- Think of a dispatch function/dictionary as an object before objects: message = method name, local state = instance state.
- Use constraint networks when a relation (like `9*c = 5*(f-32)`) should compute in any direction: setting one connector awakens constraints, which poll connectors and set others, propagating through the network.

## Anti-patterns
- **Forgetting aliasing**: mutating `suits` changes `chinese` when both names refer to the same list; with nested shared structure, `nest[0].pop(2)` changes `suits`.
- **Confusing `is` with `==`**: equal contents do not imply identity; two accounts with the same balance are different objects.
- **Assigning to a nonlocal name without declaring it**: raises `UnboundLocalError` before the function body runs.
- **Relying on dictionary order**: dictionaries are unordered collections (per this text); don't depend on printed key order.
- **Treating mutable compound data as the totality of its pieces**: valid only without mutation; with change, identity matters.

## Code Examples
```python
def make_withdraw(balance):
    """Return a withdraw function that draws down balance with each call."""
    def withdraw(amount):
        nonlocal balance                 # Declare the name "balance" nonlocal
        if amount > balance:
            return 'Insufficient funds'
        balance = balance - amount       # Re-bind the existing balance name
        return balance
    return withdraw

def account(initial_balance):            # dispatch dictionary
    def deposit(amount):
        dispatch['balance'] += amount
        return dispatch['balance']
    def withdraw(amount):
        if amount > dispatch['balance']:
            return 'Insufficient funds'
        dispatch['balance'] -= amount
        return dispatch['balance']
    dispatch = {'deposit': deposit, 'withdraw': withdraw,
                'balance': initial_balance}
    return dispatch
```
- **What it demonstrates**: local state via nonlocal, then the dispatch-dictionary pattern that stores state beside behavior (no nonlocal needed).

## Worked Example
The constraint system. Celsius/Fahrenheit relation `9*c = 5*(f-32)` is built as a network: `converter(c, f)` wires two `multiplier`s, an `adder`, and three `constant`s through internal connectors. `adder(a,b,c) = make_ternary_constraint(a,b,c, add, sub, sub)`; `multiplier` uses `mul, truediv, truediv`. A constraint's `new_val` response polls which two connectors have values and sets the third; its `forget` response tells all its connectors to forget. A connector (closure-based dispatch dictionary) stores `val`, remembers its `informant`, notifies all constraints except the source (`inform_all_except`), and reports "Contradiction detected" if set to a conflicting value. Session: `celsius['set_val']('user', 25)` propagates to `Fahrenheit = 77.0`; setting fahrenheit to 212 raises a contradiction until `celsius['forget']('user')` clears the network; then setting fahrenheit to 212 computes `Celsius = 100.0` — the same network runs both directions. This architecture of dictionaries with string keys and functional values prefigures the object system of 2.6.

## Key Takeaways
1. Mutation makes names co-refer meaningfully: always track which names alias the same object; use `is` for identity, `==` for equality.
2. `nonlocal` is required to rebind an enclosing-frame name; access never needs it.
3. Each call to a maker function creates an independent stateful object; assignment of the function value creates an alias, not a copy.
4. Mutation destroys referential transparency — reason about environments and frames, not substitution.
5. Message passing (dispatch functions, then dispatch dictionaries) implements full mutable abstract data types — lists and dicts can be built from functions with local state.
6. Non-directional computation (constraint propagation) is a distinct paradigm enabled by mutable state plus message passing.

## Connects To
- **Section 2.2**: functional pairs extend to functional *mutable* lists — data is still behavior.
- **Section 2.5**: objects and classes are the language-supported form of dispatch with local state.
- **Section 2.6**: the object system implementation reuses the dispatch-dictionary architecture directly.
