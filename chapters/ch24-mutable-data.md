# Section 2.4: Mutable Data

## Core Idea
Effective programming requires organizational principles that make large systems modular — dividing naturally into coherent parts that can be separately developed and maintained. One powerful technique is to incorporate data that may change state over time: a single data object then represents something that evolves independently of the rest of the program, and its behavior may be influenced by its history, just like an entity in the world. Adding state to data is a central ingredient of the paradigm called object-oriented programming.

## Frameworks Introduced
- **The object metaphor** (2.4.1): objects combine data values with behavior. They represent information, but also *behave* like the things they represent — an object knows how to print itself, how to reveal its parts on demand. Objects are both information and processes, bundled together to represent the properties, interactions, and behaviors of complex things. All values in Python are objects.
- **Mutable vs immutable** (2.4.2): instances of primitive built-in values such as numbers are immutable — the values themselves cannot change over program execution. Lists are mutable; mutation operations change the value of the list, they do not create new list objects. Mutable objects represent values that change over time (a person is the same person despite having aged or received a haircut).
- **Identity vs equality**: `is` and `is not` test whether two expressions in fact evaluate to the identical object. Two objects are identical if they are equal in their current value *and* any change to one will always be reflected in the other. Identity is a stronger condition than equality (`==` checks equality of contents only).
- **Local state** (2.4.4): lists and dictionaries have local state — they are changing values with some particular contents at any point in execution. Functions can also have local state, via `nonlocal` bindings in their parent frame.
- **The nonlocal statement**: lets a function *change* (not merely access) a binding in an enclosing frame. Non-local assignment is the key step toward viewing a program as a collection of independent, autonomous objects that interact but each manage their own internal state.
- **Referential transparency and its loss** (2.4.6): an expression containing only pure function calls is referentially transparent — its value does not change if a subexpression is replaced by its value. Re-binding operations violate referential transparency because they do more than return a value; they change the environment.
- **Message passing** (2.4.7): encapsulate the logic for all operations on a data value within one *dispatch function* that responds to different messages — strings, passed as the first argument, that correspond to particular behaviors; additional arguments parameterize the behavior. Dispatch functions are effectively many functions in one.
- **Dispatch dictionaries** (2.4.8): replace conditional dispatch with a dictionary keyed by message strings. Storing local state in the dictionary itself (alongside the functions) avoids the need for `nonlocal`.
- **Constraint propagation / declarative programming** (2.4.9): declare the structure of a problem as relations among quantities — a network of primitive constraints joined by connectors — and abstract away how the solution is computed. The same network computes in multiple directions; this non-directionality is the distinguishing feature of constraint-based systems.

## Evaluation Rules
The author's formal rules, stated faithfully:

1. **Attribute access**: `<expression>.<name>` — the expression evaluates to an object, and the name is an attribute of that object. Unlike ordinary names, attribute names are not available in the general environment; they are particular to the object instance preceding the dot.
2. **Methods**: function-valued attributes. By implementation, methods are functions that compute their results from both their arguments and their object (`tues.strftime('%A')` needs the format string and the date bundled into `tues`).
3. **Mutation does not create objects**: list mutation operations (`pop`, `remove`, `append`, `extend`, `insert`, element assignment, slice assignment) change the value of the list; they do not create new list objects. Hence methods called on one name can affect another name at the same time — new behavior relative to pure programs, where a name not appearing in a statement could not be affected by it.
4. **Identity rule**: two objects are identical (`is`) iff they are equal in their current value and any change to one will always be reflected in the other. `==` checks equality of contents; `is` checks identity, the stronger condition.
5. **nonlocal semantics**: `nonlocal balance` declares that whenever the binding of `balance` is changed, it is changed in *the first frame in which `balance` is already bound*. Without `nonlocal`, assignment always binds a name in the first (local) frame of the current environment. The statement indicates the name appears somewhere other than the first (local) frame or the last (global) frame. If the name has not previously been bound to a value, the `nonlocal` statement gives an error. After `nonlocal balance` executes, *all remaining* assignments to `balance` in that function re-bind it in the enclosing frame.
6. **Access never needs nonlocal**: a locally defined function can always look up names outside its local frame; only *changing* their bindings requires `nonlocal`.
7. **Python's same-frame restriction**: within the body of a function, all instances of a name must refer to the same frame. Python cannot look up a name in a non-local frame and also bind that same name in the local frame. This lets Python pre-compute which frame contains each name before executing the body; violating it produces `UnboundLocalError` *before* the offending line runs. (Python particular: most languages with higher-order functions and lexical scope make non-local assignment the default and need no statement at all. `nonlocal` did not exist in Python 2.)
8. **Frame-analysis rule**: only function calls can introduce new frames; assignment statements always change bindings in *existing* frames. Unless `make_withdraw` is called twice, there can be only one binding for `balance`.
9. **Sameness and change**: so long as we never modify data objects, a compound data object is precisely the totality of its pieces (a rational number is determined by its numerator and denominator). In the presence of change, a compound data object has an *identity* that is something different from the pieces of which it is composed: a bank account is still "the same" account after a withdrawal; two accounts can happen to have the same balance yet be different objects. Two separately defined functions are not the same, because changes to one may not be reflected in the other.
10. **Dictionary restrictions**: (a) a key cannot be or contain a mutable value — intuitively, the key tells Python where to find the pair in memory, and if the key changes, the location may be lost (tuples are commonly used as keys because lists cannot be); (b) there can be at most one value for a given key — a consequence of the abstraction, which retrieves *the* value for a key. Dictionaries are unordered collections: the printed order of keys cannot be predicted and may change between runs (per this text).
11. **Dual roles of assignment**: with `nonlocal`, assignment either changes local bindings or changes nonlocal bindings. Assignment already had dual roles — creating new bindings vs re-binding existing names — and can also change the contents of lists and dictionaries. These many roles can obscure effects; it is up to the programmer to document code clearly.

## Key Concepts
- **Object**: information plus behavior, bundled; constructed by calling a class (`date(2014, 5, 13)`); an *instance* of its class.
- **Attribute / method**: named value that is part of an object (`tues.year`); a method is a function-valued attribute computing from its arguments and its object.
- **Sharing**: two names bound to the same list co-refer (`suits = chinese`); mutation through either name is visible through both. `list(s)` makes a copy — changes to one list do not affect another *unless they share structure* (a nested shared element still aliases).
- **Tuple**: immutable sequence, comma-separated literal with optional parentheses; `()` empty, `(10,)` one element; supports `len`, selection, `count`, `index`, `+`, `*`, but no content-mutating methods; may contain mutable elements whose values can still change (`nest = (10, 20, [30, 40]); nest[2].pop()`); used implicitly in multiple assignment.
- **Dictionary**: key-value store; lookup with `numerals['X']`; add/change via assignment; `keys`/`values`/`items` return iterables; `dict([(3, 9), (4, 16)])` constructor; `get(key, default)`; comprehension `{x: x*x for x in range(3,6)}` creates a new dictionary.
- **Local state**: changing value with particular contents at each point in execution; functions acquire it through parent-frame bindings mutated with `nonlocal`.
- **Dispatch function**: a function whose first argument is a message string naming the behavior; many functions in one, possibly with local state.
- **Message passing**: the discipline of organizing computation by passing messages as first arguments to dispatch functions.
- **Dispatch dictionary**: dictionary mapping message strings to functions and data; the dictionary is itself the object.
- **Constraint**: dictionary responding to `new_val` and `forget`; holds no local state itself; its responses are non-pure functions that change the connectors it constrains.
- **Connector**: object that "holds" a value and may participate in one or more constraints; responds to `set_val`, `has_val`, `val`, `forget`, `connect`.
- **Informant**: the source (constraint or `'user'`) a connector remembers as having set its value; only that source's retraction makes it forget.

## Worked Examples

### 1. `make_withdraw` — local state via nonlocal
Goal: successive calls `withdraw(25) → 75`, `withdraw(25) → 50`, `withdraw(60) → 'Insufficient funds'`, `withdraw(15) → 35` from a starting balance of 100. `withdraw(25)` evaluated twice yields different values — the function is non-pure; calling it has the side effect of changing a name-value binding outside the current frame.

```python
def make_withdraw(balance):
    """Return a withdraw function that draws down balance with each call."""
    def withdraw(amount):
        nonlocal balance                 # Declare the name "balance" nonlocal
        if amount > balance:
            return 'Insufficient funds'
        balance = balance - amount       # Re-bind balance in the first non-local frame of the current environment
        return balance
    return withdraw
```

Trace of `wd = make_withdraw(20); wd(5); wd(3)`:
- The `def` binds `make_withdraw` globally. Calling it creates a frame binding `balance = 20` and returns the locally defined `withdraw`, whose parent is that frame. Crucially, there will be only this *single* binding for `balance` throughout the example.
- `wd(5)`: a new frame extends the `make_withdraw` frame. `nonlocal balance` means the assignment re-binds `balance` to 15 in the parent frame, not locally. Return 15.
- `wd(3)`: a *second* local frame, but with the *same parent*. It sees `balance = 15`, re-binds to 12, returns 12 — the change from the first call affects the second. Calling `withdraw` alters the environment that future calls will extend.

Independence vs aliasing:
- `wd = make_withdraw(20); wd2 = make_withdraw(7)` — two calls, two frames, two `balance` bindings. `wd2(6)` does not affect `wd`; its balance is still 20. Each instance manages its own inaccessible state: an abstraction of a bank account that changes over time based on its own history of withdrawal requests.
- `wd = make_withdraw(12); wd2 = wd; wd2(1); wd(1)` — one object, two names. Calling `wd2` *does* change the function named `wd`, because both names refer to the same function.

### 2. Dispatch functions and the dispatch-dictionary account
First, mutable lists and dictionaries can themselves be built from functions with local state — proving functions with `nonlocal` state suffice for mutable abstract data types equivalent to built-in `list` and `dict`. A mutable linked list is a dispatch function (it needs an identity: `None` cannot represent an empty mutable list because `None is None`, but two distinct functions each holding `empty` as local state are distinguishable):

```python
def mutable_link():
    """Return a functional implementation of a mutable linked list."""
    contents = empty
    def dispatch(message, value=None):
        nonlocal contents
        if message == 'len':
            return len_link(contents)
        elif message == 'getitem':
            return getitem_link(contents, value)
        elif message == 'push_first':
            contents = link(value, contents)
        elif message == 'pop_first':
            f = first(contents)
            contents = rest(contents)
            return f
        elif message == 'str':
            return join_link(contents, ", ")
    return dispatch

def to_mutable_link(source):
    """Return a functional list with the same contents as source."""
    s = mutable_link()
    for element in reversed(source):
        s('push_first', element)
    return s
```

Usage: `s = to_mutable_link(suits); type(s)` is `<class 'function'>`; `s('str')` returns `'heart, diamond, spade, club'`; `s('pop_first')` returns `'heart'` and mutates the list. In principle `push_first` and `pop_first` suffice for arbitrary changes (empty the list, rebuild). A functional dictionary works the same way, storing a list of two-element `[key, value]` records; `setitem` filters out existing records with the key then appends one (so each key appears once); `getitem` filters for the match:

```python
def dictionary():
    """Return a functional implementation of a dictionary."""
    records = []
    def getitem(key):
        matches = [r for r in records if r[0] == key]
        if len(matches) == 1:
            key, value = matches[0]
            return value
    def setitem(key, value):
        nonlocal records
        non_matches = [r for r in records if r[0] != key]
        records = non_matches + [[key, value]]
    def dispatch(message, key=None, value=None):
        if message == 'getitem':
            return getitem(key)
        elif message == 'setitem':
            setitem(key, value)
    return dispatch
```

(Not optimized — each call filters all records; the built-in dict is considerably more efficient, implementation out of scope.)

The dispatch *dictionary* refines the pattern: instead of conditionals, use a dictionary with string keys. State lives in the dictionary alongside behavior, so `deposit`/`withdraw` need no `nonlocal` — they mutate `dispatch['balance']`:

```python
def account(initial_balance):
    def deposit(amount):
        dispatch['balance'] += amount
        return dispatch['balance']
    def withdraw(amount):
        if amount > dispatch['balance']:
            return 'Insufficient funds'
        dispatch['balance'] -= amount
        return dispatch['balance']
    dispatch = {'deposit':   deposit,
                'withdraw':  withdraw,
                'balance':   initial_balance}
    return dispatch

def withdraw(account, amount):
    return account['withdraw'](amount)
def deposit(account, amount):
    return account['deposit'](amount)
def check_balance(account):
    return account['balance']

a = account(20)
deposit(a, 5)      # 25
withdraw(a, 17)    # 8
check_balance(a)   # 8
```

(`+=` / `-=` are shorthand for combined lookup and re-assignment: `a = a + 1` and `a += 1` are equivalent.)

### 3. Constraint propagation — the celsius/fahrenheit converter
The relation `9 * c = 5 * (f - 32)` is one constraint, built as a network of primitive `adder`, `multiplier`, and `constant` constraints joined by connectors. Compare the ideal gas law `p * v = n * k * t`: given any four quantities we can compute the fifth, yet a traditional function forces one direction.

**Propagation algorithm**: when a connector is given a value (by the user or a constraint), it awakens all of its associated constraints *except the one that just awakened it*. Each awakened constraint polls its connectors to see if there is enough information to determine a value for some connector; if so, it sets that connector, which awakens its constraints, and so on. In the converter, constants immediately set `w, x, y` to 9, 5, 32. Setting `celsius` to 25 awakens the leftmost multiplier, which sets `u = 25 * 9 = 225`; `u` awakens the second multiplier, which sets `v = 45`; `v` awakens the adder, which sets `fahrenheit = 77`.

```python
def converter(c, f):
    """Connect c to f with constraints to convert from Celsius to Fahrenheit."""
    u, v, w, x, y = [connector() for _ in range(5)]
    multiplier(c, w, u)
    multiplier(v, x, u)
    adder(v, y, f)
    constant(w, 9)
    constant(x, 5)
    constant(y, 32)

celsius = connector('Celsius')
fahrenheit = connector('Fahrenheit')
converter(celsius, fahrenheit)
```

Session (named connectors print changes):
```python
>>> celsius['set_val']('user', 25)
Celsius = 25
Fahrenheit = 77.0
>>> fahrenheit['set_val']('user', 212)
Contradiction detected: 77.0 vs 212
>>> celsius['forget']('user')
Celsius is forgotten
Fahrenheit is forgotten
>>> fahrenheit['set_val']('user', 212)
Fahrenheit = 212
Celsius = 100.0
```
The forget request succeeds because the `'user'` is the informant that set the value originally; the retraction propagates, `fahrenheit` finds it has no reason for continuing to believe its own value is 77 and gives it up. The *same* network computes celsius from fahrenheit and vice versa.

**Constraints** are dictionaries without local state; responses are non-pure functions that change connectors by sending messages (never directly), so the connector can notify others. An adder must also specify its inverses for multidirectional propagation:

```python
from operator import add, sub
def adder(a, b, c):
    """The constraint that a + b = c."""
    return make_ternary_constraint(a, b, c, add, sub, sub)

def make_ternary_constraint(a, b, c, ab, ca, cb):
    """The constraint that ab(a,b)=c and ca(c,a)=b and cb(c,b) = a."""
    def new_value():
        av, bv, cv = [connector['has_val']() for connector in (a, b, c)]
        if av and bv:
            c['set_val'](constraint, ab(a['val'], b['val']))
        elif av and cv:
            b['set_val'](constraint, ca(c['val'], a['val']))
        elif bv and cv:
            a['set_val'](constraint, cb(c['val'], b['val']))
    def forget_value():
        for connector in (a, b, c):
            connector['forget'](constraint)
    constraint = {'new_val': new_value, 'forget': forget_value}
    for connector in (a, b, c):
        connector['connect'](constraint)
    return constraint

from operator import mul, truediv
def multiplier(a, b, c):
    """The constraint that a * b = c."""
    return make_ternary_constraint(a, b, c, mul, truediv, truediv)

def constant(connector, value):
    """The constraint that connector = value."""
    constraint = {}
    connector['set_val'](constraint, value)
    return constraint
```

The `constraint` dictionary is a dispatch dictionary *and* the constraint object itself: it responds to `new_val`/`forget` but is also passed as the `source` argument to its connectors. On `forget`, it asks all its connectors to forget — but only values set *by this constraint* are actually lost (the informant check). A constant is a constraint that is never sent messages: it sets its single connector on construction.

**Connectors** are dispatch dictionaries with closure-held local state (`informant`, `constraints` list); four responses are functions and the fifth (`'val'`) is the value itself:

```python
def connector(name=None):
    """A connector between constraints."""
    informant = None
    constraints = []
    def set_value(source, value):
        nonlocal informant
        val = connector['val']
        if val is None:
            informant, connector['val'] = source, value
            if name is not None:
                print(name, '=', value)
            inform_all_except(source, 'new_val', constraints)
        else:
            if val != value:
                print('Contradiction detected:', val, 'vs', value)
    def forget_value(source):
        nonlocal informant
        if informant == source:
            informant, connector['val'] = None, None
            if name is not None:
                print(name, 'is forgotten')
            inform_all_except(source, 'forget', constraints)
    connector = {'val': None,
                 'set_val': set_value,
                 'forget': forget_value,
                 'has_val': lambda: connector['val'] is not None,
                 'connect': lambda source: constraints.append(source)}
    return connector

def inform_all_except(source, message, constraints):
    """Inform all constraints of the message, except source."""
    for c in constraints:
        if c != source:
            c[message]()
```

`set_value` sets the value and remembers the requesting source as `informant` only when the connector has no value; otherwise a differing value prints a contradiction. `forget_value` releases the value only if the request comes from the same constraint that set it originally, then propagates. The constraint program introduces ideas that return in OOP: constraints and connectors are abstractions manipulated through messages, and setting a value both validates it (checking the source) and propagates its effects. The same architecture — dictionaries with string-valued keys and functional values — is used to implement an object system later in the chapter.

## Reference Tables

Mutation operations on lists (the playing-cards history example):

| Operation | Effect |
|---|---|
| `suits.pop()` | Remove and return the final element |
| `suits.remove('string')` | Remove the first element that equals the argument |
| `suits.append('cup')` | Add an element to the end |
| `suits.extend(['sword', 'club'])` | Add all elements of a sequence to the end |
| `suits[2] = 'spade'` | Replace an element |
| `suits[0:2] = ['heart', 'diamond']` | Replace a slice |
| `suits.insert(2, 'Joker')` | Insert an element at index 2, shifting the rest |
| also | methods for sorting and reversing |

Identity vs equality:

| Expression | Result | Why |
|---|---|---|
| `suits is nest[0]` | `True` | same object (nest[0] was assigned `suits`) |
| `suits is ['heart', 'diamond', 'spade', 'club']` | `False` | equal contents, different list object |
| `suits == ['heart', 'diamond', 'spade', 'club']` | `True` | `==` checks equality of contents |

Connector messages:

| Message | Meaning |
|---|---|
| `connector['set_val'](source, value)` | source requests the connector set its current value to value |
| `connector['has_val']()` | whether the connector already has a value |
| `connector['val']` | the current value of the connector (data, not a function) |
| `connector['forget'](source)` | source requests the connector forget its value |
| `connector['connect'](source)` | participate in a new constraint, the source |

Constraint messages:

| Message | Meaning |
|---|---|
| `constraint['new_val']()` | some connected connector has a new value |
| `constraint['forget']()` | some connected connector has forgotten its value |

Ternary constraint instantiations:

| Constructor | Relation | ab | ca | cb |
|---|---|---|---|---|
| `adder(a, b, c)` | `a + b = c` | `add` | `sub` | `sub` |
| `multiplier(a, b, c)` | `a * b = c` | `mul` | `truediv` | `truediv` |

## Mental Models
- **Frames, not substitution**: to analyze code with non-local assignment, remember that only function calls introduce new frames; assignment statements always change bindings in existing frames. Count calls to the maker function to count state bindings.
- **Closure = object**: each call to `make_withdraw` produces an independent stateful instance (own parent frame, own `balance`); assigning the function value (`wd2 = wd`) creates an alias to one object, not a copy. It is not unusual for two names to co-refer in the world, and so in programs — but with changing values you must track the effect of a change on every co-referring name.
- **Message = method before methods**: a dispatch function or dispatch dictionary is an object system in miniature — message string as method name, closure or dictionary slots as instance state. The choice between adding `elif` clauses to `dispatch` versus building new operations from existing messages mirrors method-vs-helper design.
- **Constraint network = non-directional program**: when a relation like `9*c = 5*(f-32)` should compute in any direction, wire connectors through constraints instead of writing one-directional functions. Values propagate by awakening; retraction propagates by forgetting, gated by the informant.
- **Pre-computation intuition**: Python considers assignment targets throughout a function body before executing it — that is why `UnboundLocalError` fires before the assignment line runs. Pre-computing facts about a function body before execution is common in interpreter design.

## Common Errors
Mistakes the author explicitly demonstrates or warns about:

1. **Assigning to an enclosing-frame name without `nonlocal`** — `UnboundLocalError: local variable 'balance' referenced before assignment`. Because `balance` is assigned somewhere in the body, Python assumes all references to it are local; the error occurs *before* the assignment line executes. Adding `nonlocal balance` corrects it. "When this restriction is violated, a confusing error message results."
2. **`nonlocal` for an unbound name** — if the name has not previously been bound in an enclosing frame, the `nonlocal` statement gives an error. (It also cannot target the local or global frame.)
3. **Forgetting aliasing** — after `suits = chinese`, every mutation through `suits` changes what `chinese` names; previously a name absent from a statement could not be affected by it. With shared structure, `nest[0] = suits` means `suits.insert(2, 'Joker')` appears inside `nest`, and `nest[0].pop(2)` changes `suits`.
4. **Confusing copies with aliases** — `list(suits)` copies, so changes to one list do not affect the other *unless they share structure*; an element that is itself a shared list still aliases.
5. **Confusing `is` with `==`** — equal contents never imply identity; two bank accounts with the same balance are different objects. Conversely `None is None`, which is why `None` cannot represent an empty *mutable* list — mutable values need identity.
6. **Expecting tuples to be fully frozen** — you cannot change which elements are in a tuple, but a mutable element contained within it can still change (`nest[2].pop()` on `(10, 20, [30, 40])`).
7. **Using a mutable value as (or inside) a dictionary key** — forbidden; if the key changes, the location of the pair may be lost. Use tuples, not lists.
8. **Relying on dictionary order** — dictionaries are unordered collections in this text; printed order is unpredictable and may change across runs (`'L'` was not added at the end of the printed `numerals`).
9. **Assuming aliasing-by-assignment copies state** — `wd2 = wd` is one function with two names; calling `wd2` changes `wd`'s future results. Only a second call to `make_withdraw` makes a second `balance`.
10. **Undocumented assignment effects** — assignment now variously creates bindings, re-binds names locally or nonlocally, and mutates list/dict contents; the many roles can obscure what a statement does, so document clearly.

## Anti-patterns
- **Treating mutable compound data as the totality of its pieces**: valid only while nothing is modified; under change, identity is distinct from contents.
- **One-directional functions for inherently relational problems**: translating `p * v = n * k * t` into a function for pressure makes it useless for temperature, though both arise from the same equation — reach for constraints instead.
- **Letting constraints mutate connector values directly**: constraints must change connectors only by sending messages, so the connector can validate (check the source) and notify the rest of the network.
- **Reaching for `nonlocal` when a dispatch dictionary stores state**: keeping `balance` in the dictionary alongside the functions removes the need for `nonlocal` entirely.

## Key Takeaways
1. Mutation makes co-reference meaningful: track which names alias one object; use `is` for identity, `==` for contents. Identity is the stronger condition.
2. `nonlocal` is required to re-bind an enclosing-frame name; access never needs it. Python's one-frame-per-name rule means a missing `nonlocal` fails before the body runs.
3. Only function calls introduce frames; assignments mutate existing ones. One maker call = one state binding; function assignment aliases, it does not copy.
4. Non-local assignment buys modularity (independent autonomous objects, state inaccessible to the rest of the program) at the cost of referential transparency — reason with environments, not substitution.
5. Functions with local state can implement mutable abstract data types equivalent to built-in `list` and `dict` — message passing via dispatch functions, then dispatch dictionaries.
6. Constraint propagation shows mutable state plus message passing enabling a new paradigm: declarative, non-directional computation with contradiction detection and informant-gated retraction.

## Connects To
- **Section 1.6 / nested def**: closures could always *read* enclosing names; this section adds the power to change them.
- **Section 2.2–2.3**: the functional pairs and linked lists of data abstraction extend to functional *mutable* lists — data is still behavior.
- **Section 2.5**: constraints and connectors manipulated through messages prefigure objects and classes, the language-supported form of dispatch with local state.
- **Section 2.6**: the object system is implemented with the same architecture — dictionaries with string-valued keys and functional values.
- **Interpreter chapters**: Python's pre-computation of name frames is an early instance of analyzing a function body before executing it.
