# Section 2.6: Implementing Classes and Objects

## Core Idea
Classes and objects can be represented with nothing but functions and dictionaries: the object metaphor does not require a special language. Programs can be object-oriented even in languages without a built-in object system — instances, classes, and base classes are just dispatch dictionaries exchanging messages.

## Frameworks Introduced
- **Instances as dispatch dictionaries**: an instance responds to `get` and `set`. `set` writes to a local `attributes` dict (attribute assignment always stays local to the object); `get` returns a local attribute if present, otherwise asks the class and, if the result is a function, binds it to the instance.
  - When to use: to understand what Python's dot notation actually does, or to build OOP in a non-OO language.
- **bind_method**: `bind_method(value, instance)` returns `lambda *args: value(instance, *args)` if `value` is callable, else `value` unchanged — this is where `self` binding comes from.
- **Classes as dispatch dictionaries**: a class responds to `get`, `set`, and `new`. Class `get` falls back to the **base_class** (not to a class of its own — classes here have no class); no method binding at the class level.
- **init_instance**: the `new` message makes an instance via `make_instance(cls)`, looks up `'__init__'` in the class, and calls it on the fresh instance with the constructor args.
- **Single user entry point**: in this system the only function the user should call is `make_class`; everything else happens through message passing — just as Python's object system is invoked by the `class` statement and driven by dot expressions and class calls.

## Key Concepts
- **Attribute lookup chain**: instance attributes → class attributes → base-class attributes (recursive), mirroring the dot-expression procedure of 2.5.
- **locals()**: returns a dictionary of the current frame's name-value bindings; `make_class(locals())` turns a function body full of defs and assignments into a class — the analogue of a class suite.
- **Overriding in this system**: a subclass dict simply contains its own `withdraw`; ancestor calls go through the base class object: `Account['get']('withdraw')(self, amount + fee)`.
- **Instance/class separation**: `instance['set']` never touches the class — setting `interest` on `kirk_account` leaves `Account['get']('interest')` at 0.02.
- **__dict__ parallel**: real Python instances store local attributes in `__dict__`, much like this system's `attributes` dictionary; Python additionally distinguishes special methods that integrate with built-in functions (subject of 2.7).
- **Scope of the implementation**: user-defined classes without multiple inheritance or introspection; not meta-classes or static methods.

## Mental Models
- Think of dot notation as sugar: `obj.name` ≈ `obj['get']('name')`; `obj.name = v` ≈ `obj['set']('name', v)`; `Cls(args)` ≈ `Cls['new'](args)`.
- Think of a bound method as a closure over the instance, manufactured at lookup time by `get`.
- Remember the tower of implementations: dictionaries were built on lists, lists on pairs, pairs on functions — so this object system could run on functions alone.

## Anti-patterns
- **Expecting class `get` to bind methods**: binding happens only on instance lookup; a function fetched from a class is a plain function (matching `type(Account.deposit)` being `function` in real Python).
- **Confusing attribute set with class modification**: setting through an instance can never change the class, by construction.

## Code Examples
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
- **What it demonstrates**: the complete object system in four functions — lookup fallback, self-binding, inheritance fallback, and construction.

## Worked Example
Rebuilding Account. `make_account_class()` defines `interest = 0.02`, `__init__`, `deposit`, `withdraw` as ordinary local names (methods read/write state via `self['get']`/`self['set']`), then returns `make_class(locals())`. `Account = make_account_class()`; `kirk_account = Account['new']('Kirk')`; `kirk_account['get']('deposit')(20)` → 20; `kirk_account['get']('withdraw')(5)` → 15. Setting instance interest to 0.04 leaves the class at 0.02. `make_checking_account_class()` defines `interest = 0.01`, `withdraw_fee = 1`, and a `withdraw` that fetches the fee via `self['get']('withdraw_fee')` and delegates to `Account['get']('withdraw')(self, amount + fee)`, returning `make_class(locals(), Account)`. The subclass instance pays the $1 fee and sees the lower interest — behavior identical to Python's built-in system of 2.5.

## Key Takeaways
1. An object system is ~40 lines of functions and dictionaries: instances (get/set + bound methods), classes (get/set/new + base fallback), and init.
2. `self` binding is nothing magical — a closure inserting the instance as first argument when a callable is fetched from the class via an instance.
3. Attribute assignment is always local to the object; lookup falls back up the chain.
4. `make_class(locals())` shows a class suite is just a frame of bindings turned into a dictionary.
5. OOP is a discipline, not a language feature — you can apply the object metaphor anywhere message passing works.

## Connects To
- **Section 2.4**: directly reuses dispatch dictionaries and message passing.
- **Section 2.5**: implements exactly the dot-expression and inheritance semantics described there.
- **Section 2.7**: real Python adds special methods so built-in functions work across types — the next layer of object abstraction.
