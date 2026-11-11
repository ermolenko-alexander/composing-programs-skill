# Section 2.6: Implementing Classes and Objects

## Core Idea
Classes and objects can be represented with nothing but functions and dictionaries: the object metaphor does not require a special programming language. Programs can be object-oriented even in languages without a built-in object system — instances, classes, and base classes are all just dispatch dictionaries exchanging messages. The implementation abandons dot notation (which does require built-in language support) but reproduces the behavior of Python's object system: local attribute storage, class fallback on lookup, method binding, inheritance through a base class, and construction via `__init__`.

The section deliberately does *not* implement the entire Python object system. Out of scope: meta-classes, static methods, multiple inheritance, and introspective behavior (such as returning the class of an instance). The goal is the core functionality that enables the object metaphor, not the precise specification of the Python type system.

## Frameworks Introduced
- **Instances as dispatch dictionaries**: an instance responds to `get` and `set` messages. `set` writes to a local `attributes` dict (attribute assignment always stays local to the object); `get` returns a local attribute if present, otherwise asks the class and, if the returned value is a function, binds it to the instance.
  - When to use: to understand what Python's dot notation actually does, or to build OOP in a non-OO language.
- **bind_method**: creates a bound method value from a function value by inserting the instance as the first argument. Applies only to callable values; non-callable values pass through unchanged. This is where `self` comes from.
- **Classes as dispatch dictionaries**: a class responds to `get`, `set`, and `new`. For simplicity, classes do not themselves have a class (in Python they do — almost all share the class `type`). Class `get` falls back to the **base_class**, not to a class of its own, and performs no method binding.
- **init_instance**: the `new` message makes an instance via `make_instance(cls)`, looks up `'__init__'` through the class's `get` (so an inherited constructor works), and if found calls it on the fresh instance with the constructor args.
- **Single user entry point**: the only function the user should call is `make_class`; all other functionality is enabled through message passing — just as Python's object system is invoked via the `class` statement and driven by dot expressions and calls to classes.
- **`locals()` as class suite**: `make_class(locals())` turns a function body full of `def`s and assignments into a class dictionary; `locals` returns a dictionary with string keys containing the name-value bindings of the current local frame.

## Evaluation Rules
The author's procedures, stated as rules. These exactly mirror the dot-expression evaluation procedure of Section 2.5.

**Instance `get(name)`** (attribute lookup on an instance):
1. If `name` appears in the instance's local `attributes` dictionary, return `attributes[name]`.
2. Otherwise, look `name` up in the class: `value = cls['get'](name)`.
3. Return `bind_method(value, instance)` — binding applies only when the class-supplied value is a function.

**Instance `set(name, value)`** (attribute assignment):
- `attributes[name] = value`. Always. Assignment is stored directly in the object's local attribute dictionary and never touches the class. This corresponds to attribute assignment in Python's object system.

**bind_method(value, instance)**:
- If `callable(value)`: return `method` where `method(*args)` computes `value(instance, *args)` — when the method is called, the first parameter `self` is bound to the instance.
- Otherwise: return `value` unchanged.

**Class `get(name)`** (class attribute lookup):
1. If `name` is in the class's `attributes` dictionary, return it.
2. Otherwise, if `base_class is not None`, return `base_class['get'](name)` — recursion walks the inheritance chain.
3. (Implicitly, if there is no base class and the name is absent, the result is `None`.)

Key contrasts with instance `get`: a class does not query its class (it has none here) — it queries its base class; and **no method binding happens at the class level**.

**Class `set(name, value)`**: writes into the class's own `attributes` dictionary.

**Class `new(*args)`** → **init_instance(cls, \*args)**:
1. `instance = make_instance(cls)` — a fresh instance whose class is `cls`.
2. `init = cls['get']('__init__')` — looked up through the class, so it may come from a base class.
3. If `init` is truthy, call `init(instance, *args)` — the raw function from the class dict, called with the instance explicitly as first argument (no binding needed; `init_instance` supplies it directly).
4. Return the instance.

**Full attribute lookup chain**: instance `attributes` → class `attributes` → base-class `attributes` → ... (recursively), with method binding applied once, at the instance level, to whatever callable comes back.

## Key Concepts
- **Dispatch dictionary**: a dictionary mapping message names (`'get'`, `'set'`, `'new'`) to functions that close over local state — the mechanism from 2.4.8, now layered into a full object system.
- **Instance**: dispatch dictionary with `get`/`set`; state lives in a closed-over `attributes` dict.
- **Class**: dispatch dictionary with `get`/`set`/`new`; shared attributes and methods live in its `attributes` dict (the one passed to `make_class`).
- **Bound method value**: a closure manufactured at lookup time that inserts the instance as the first argument of a class function.
- **base_class**: optional second parameter to `make_class`; the inheritance link, consulted by class `get` on a miss.
- **`__init__`**: the constructor method, invoked by `init_instance` immediately after instance creation.
- **`locals()`**: returns the current frame's bindings as a dict — the analogue of a class suite's namespace.
- **Attribute lookup chain**: instance → class → base class (recursive), mirroring 2.5's dot-expression procedure.
- **Overriding**: a subclass dict simply contains its own version of a name; it shadows the base class in `get` fallback.
- **Ancestor (base-class) call**: fetch the raw function from the base class object and pass `self` explicitly: `Account['get']('withdraw')(self, amount + fee)`.
- **`__dict__` parallel**: in Python, an instance of any user-defined class has a special attribute `__dict__` storing its local instance attributes — much like this system's `attributes` dictionary.
- **Tower of implementations**: dictionaries were implemented with lists, lists with pairs, pairs with functions — so this object system could just as well be implemented using functions alone.

## Mental Models
- Dot notation is sugar over message passing: `obj.name` ≈ `obj['get']('name')`; `obj.name = v` ≈ `obj['set']('name', v)`; `Cls(args)` ≈ `Cls['new'](args)`.
- A bound method is a closure over the instance, manufactured fresh on every instance-level `get` of a callable class attribute.
- A class statement is a frame of bindings turned into a dictionary: define names in a function body, return `make_class(locals())`.
- Two parallel systems, one semantics: every behavior demonstrated here (local sets, class fallback, binding, inheritance, constructor) is behavior Python's built-in system exhibits — the implementation explains the language feature.

## Reference Table: Message Protocol

| Message | Instance | Class |
|---|---|---|
| `get(name)` | local `attributes`, else class `get`, then `bind_method` | own `attributes`, else `base_class['get']` (no binding) |
| `set(name, value)` | writes local `attributes` only | writes class `attributes` |
| `new(*args)` | — (not supported) | `init_instance(cls, *args)` |
| Fallback target | its class | its base class (classes have no class here) |
| Method binding | yes, on callable values from class | never |

## Reference Table: Implemented System vs Python

| This system | Python's built-in system |
|---|---|
| `instance['get']('name')` | `instance.name` |
| `instance['set']('name', v)` | `instance.name = v` |
| `Cls['new'](args)` | `Cls(args)` |
| `attributes` dict in `make_instance` | instance `__dict__` |
| `make_class(locals())` at end of maker function | `class` statement suite |
| `base_class` parameter | base class in `class C(Base):` |
| classes have no class | classes have class `type` |
| no special methods | special methods integrate with built-in functions (2.7) |

## Code Examples
The complete object system — four functions.

```python
def make_instance(cls):
    """Return a new object instance, which is a dispatch dictionary."""
    def get_value(name):
        if name in attributes:
            return attributes[name]
        else:
            value = cls['get'](name)
            return bind_method(value, instance)
    def set_value(name, value):
        attributes[name] = value
    attributes = {}
    instance = {'get': get_value, 'set': set_value}
    return instance

def bind_method(value, instance):
    """Return a bound method if value is callable, or value otherwise."""
    if callable(value):
        def method(*args):
            return value(instance, *args)
        return method
    else:
        return value

def make_class(attributes, base_class=None):
    """Return a new class, which is a dispatch dictionary."""
    def get_value(name):
        if name in attributes:
            return attributes[name]
        elif base_class is not None:
            return base_class['get'](name)
    def set_value(name, value):
        attributes[name] = value
    def new(*args):
        return init_instance(cls, *args)
    cls = {'get': get_value, 'set': set_value, 'new': new}
    return cls

def init_instance(cls, *args):
    """Return a new object with type cls, initialized with args."""
    instance = make_instance(cls)
    init = cls['get']('__init__')
    if init:
        init(instance, *args)
    return instance
```

- **What it demonstrates**: the lookup fallback (instance → class), self-binding (`bind_method`), inheritance fallback (class → base class), and construction (`new` → `init_instance` → `__init__`). This completes the system: instances `set` locally but fall back to their classes on `get`; after looking up a name in its class, an instance binds itself to function values to create methods; classes create `new` instances and apply `__init__` immediately after creation.

## Worked Examples

### 1. Building the object system itself, bottom-up
Start from the assumption that a class implementation exists that can look up any names not part of the instance — `make_instance` takes that class as `cls`. The instance closes over a fresh empty `attributes` dict and exposes `get`/`set`. `get_value` first checks `attributes`; on a miss it delegates to `cls['get'](name)` and passes the result through `bind_method`, which wraps callables so that the instance becomes the first argument. Then the class side: `make_class` takes an `attributes` dict and an optional `base_class`. Its `get_value` checks its own dict, then delegates to `base_class['get']` — note no binding here, and no "class of a class". Its `new` closes over `cls` (defined just below) and forwards to `init_instance`, which makes the instance, looks up `'__init__'` through the class, and calls it with the fresh instance plus the constructor arguments. Four functions, mutually arranged so the only user-facing entry point is `make_class`.

### 2. `make_account_class` — an Account from `locals()`
```python
def make_account_class():
    """Return the Account class, which has deposit and withdraw methods."""
    interest = 0.02
    def __init__(self, account_holder):
        self['set']('holder', account_holder)
        self['set']('balance', 0)
    def deposit(self, amount):
        """Increase the account balance by amount and return the new balance."""
        new_balance = self['get']('balance') + amount
        self['set']('balance', new_balance)
        return self['get']('balance')
    def withdraw(self, amount):
        """Decrease the account balance by amount and return the new balance."""
        balance = self['get']('balance')
        if amount > balance:
            return 'Insufficient funds'
        self['set']('balance', balance - amount)
        return self['get']('balance')
    return make_class(locals())
```
The structure mirrors a Python `class` statement but concludes with `make_class(locals())` — the local frame's bindings (`interest`, `__init__`, `deposit`, `withdraw`) become the class attributes. Methods manipulate state through the instance's own message protocol (`self['get']`, `self['set']`) because `self` here is a dispatch dictionary, not a Python object.

Trace of use:
```python
>>> Account = make_account_class()
>>> kirk_account = Account['new']('Kirk')
>>> kirk_account['get']('holder')
'Kirk'
>>> kirk_account['get']('interest')
0.02
>>> kirk_account['get']('deposit')(20)
20
>>> kirk_account['get']('withdraw')(5)
15
```
Step through `kirk_account['get']('deposit')(20)`: `'deposit'` is not in the instance's `attributes`, so the class supplies the raw `deposit` function; `bind_method` wraps it so that calling the result with `(20)` actually calls `deposit(kirk_account, 20)`; inside, `self['get']('balance')` reads 0 (set by `__init__`), 20 is added and stored locally. Meanwhile `'interest'` is found in the class dict and returned unbound (it isn't callable — `bind_method` passes it through).

Instance sets never leak to the class:
```python
>>> kirk_account['set']('interest', 0.04)
>>> Account['get']('interest')
0.02
```
After the `set`, `kirk_account['get']('interest')` finds 0.04 locally and never consults the class, whose value is untouched — exactly as in Python's object system.

### 3. `make_checking_account_class` — inheritance and a base-class call
```python
def make_checking_account_class():
    """Return the CheckingAccount class, which imposes a $1 withdrawal fee."""
    interest = 0.01
    withdraw_fee = 1
    def withdraw(self, amount):
        fee = self['get']('withdraw_fee')
        return Account['get']('withdraw')(self, amount + fee)
    return make_class(locals(), Account)
```
The subclass overloads only a subset of the class attributes: a lower `interest`, a new `withdraw_fee`, and a specialized `withdraw`. The ancestor call pattern: fetch the raw `withdraw` function from the base class object (`Account['get']('withdraw')` — class-level `get`, so no binding) and pass `self` explicitly, with the fee added to the amount. This is the same shape as calling the base-class method in Python's built-in system.

```python
>>> CheckingAccount = make_checking_account_class()
>>> jack_acct = CheckingAccount['new']('Spock')
>>> jack_acct['get']('interest')
0.01
>>> jack_acct['get']('deposit')(20)
20
>>> jack_acct['get']('withdraw')(5)
14
```
Trace of the inherited machinery: `CheckingAccount['new']('Spock')` → `init_instance` looks up `'__init__'`, misses in `CheckingAccount`'s dict, falls back to `Account` — the inherited constructor sets `holder` and `balance`. `deposit` behaves identically to a plain Account (also inherited). `withdraw(5)` finds the subclass's version first: it reads `withdraw_fee` via `self['get']` (instance miss → `CheckingAccount` hit → 1, not callable so unbound), then delegates to Account's `withdraw` with `amount + fee = 6`, leaving 20 − 6 = 14. `interest` resolves in `CheckingAccount` (0.01), shadowing Account's 0.02.

## Common Errors
Mistakes the author's design explicitly guards against or warns about:
- **Expecting dot notation to work**: this implementation abandons dot notation, because dot notation requires built-in language support. Every interaction must go through `['get']`, `['set']`, `['new']`.
- **Expecting class `get` to bind methods**: no method binding is required (or performed) for classes. A function fetched from a class is a plain function; only instance-level `get` binds. This matches real Python, where a function accessed on the class is unbound.
- **Binding non-function values**: `bind_method` must apply only to callable values — wrapping a number like `interest` would break attribute reads. The `callable` check is load-bearing.
- **Thinking an instance `set` can modify the class**: by construction, `set` on an instance writes only to the local `attributes` dict; setting `interest` on `kirk_account` leaves `Account['get']('interest')` at 0.02.
- **Calling internal functions directly**: the only function the user should call is `make_class`. `make_instance`, `bind_method`, and `init_instance` are plumbing reached via messages — like Python, where everything after the `class` statement happens through dot expressions and class calls.
- **Assuming full Python semantics**: this system skips meta-classes, static methods, multiple inheritance, introspection, and special methods; don't expect `Cls['get']` on a missing name with no base class to raise — it silently returns `None` (unlike Python's `AttributeError`).
- **Confusing where `self` is supplied**: `bind_method` supplies it for ordinary method calls; `init_instance` supplies it explicitly when calling `__init__`; a base-class call supplies it by hand (`Account['get']('withdraw')(self, ...)`). Forgetting the explicit `self` in an ancestor call shifts all arguments.

## Anti-patterns
- **Treating OOP as a language feature rather than a discipline**: the whole point is that the object metaphor works wherever message passing works.
- **Duplicating base-class logic in a subclass instead of delegating**: the `CheckingAccount.withdraw` pattern — adjust the arguments, delegate to the base — keeps the override minimal.
- **Storing shared data on instances**: shared values like `interest` belong in the class dict, reached by fallback; only per-object state (`balance`, `holder`) goes through `self['set']`.

## Key Takeaways
1. An object system is ~40 lines of functions and dictionaries: instances (get/set + bound methods), classes (get/set/new + base fallback), and `init_instance`.
2. `self` binding is nothing magical — a closure inserting the instance as first argument, manufactured whenever a callable is fetched from the class through an instance.
3. Attribute assignment is always local to the object; lookup falls back up the chain instance → class → base class.
4. `make_class(locals())` shows a class suite is just a frame of bindings turned into a dictionary.
5. Classes here query their base class, never a class of their own, and never bind methods — the two asymmetries between instance `get` and class `get`.
6. The system built on dictionaries is quite similar to Python's: `attributes` ≈ `__dict__`; what Python adds beyond this is special methods that make built-in functions work across types.
7. OOP is a discipline, not a language feature — you can apply the object metaphor in any language where message passing works.

## Connects To
- **Section 2.4**: directly reuses dispatch dictionaries and message passing (2.4.8); and the tower — dictionaries on lists, lists on pairs, pairs on functions — means objects could be functions all the way down.
- **Section 2.5**: implements exactly the dot-expression evaluation procedure, class attributes, and single inheritance semantics described there; the Account/CheckingAccount behavior reproduces 2.5's examples verbatim.
- **Section 2.7**: real Python additionally distinguishes special methods that interact with built-in functions so they behave correctly across many types — the next layer of object abstraction (generic functions).
