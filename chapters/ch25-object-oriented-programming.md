# Section 2.5: Object-Oriented Programming

## Core Idea
OOP unifies the chapter's techniques: like data abstraction, classes create abstraction barriers between the use and implementation of data; like dispatch dictionaries, objects respond to behavioral requests; like mutable data structures, objects have local state not directly accessible from the global environment. Beyond convenience, the object system enables a new metaphor: a program is several independent agents interacting within the computer. Each object bundles local state and behavior, abstracting the complexity of both; useful results are computed as a consequence of objects communicating. Objects don't just pass messages — they share behavior among objects of the same type and inherit characteristics from related types.

## Frameworks Introduced
- **Class statement / constructor**: `class <name>: <suite>` — executing it creates a new class bound to `<name>` in the first frame of the current environment, then executes the suite; names bound in the suite (via `def` or assignment) become class attributes. `__init__` is the **constructor**; instantiation uses call syntax (`Account('Kirk')`): a new instance is created, then `__init__` is called with the new object bound to `self` plus the call's arguments.
- **Dot expression evaluation procedure** (`<expression>.<name>`) — the section's central formal rule; see Evaluation Rules.
- **Functions vs bound methods**: `Account.deposit` is a plain function (explicit `self`); `spock_account.deposit` is a bound method already coupled to its instance. Same function body, different value types (`type(...)` returns `function` vs `method`). `getattr`/`hasattr` are the functional equivalents of dot notation and follow the same rule.
- **Class attributes vs instance attributes**: class attributes (assigned in the class suite, outside methods) are shared by all instances; assignment through an instance creates a shadowing instance attribute; assignment through the class changes all instances without a like-named instance attribute.
- **Inheritance**: `class CheckingAccount(Account):` — subclass inherits base-class attributes and may **override** them; specify only what differs. Class lookup is recursive (see Evaluation Rules). Inheritance represents **is-a** relationships; **has-a** relationships belong in instance attributes (a bank *has* a list of accounts; neither should inherit from the other).
- **Calling ancestors**: overridden attributes are still accessible via class objects — `Account.withdraw(self, amount + self.withdraw_charge)`. Prefer `self.withdraw_charge` over `CheckingAccount.withdraw_charge` so a further subclass can override the charge and existing methods pick up the new value.
- **Object interface**: a collection of attributes and conditions on those attributes. In Python (as in Ruby and Go) any object with the appropriate names implements an interface — no declaration needed (unlike Java). Code that *uses* objects is most robust when it assumes only attribute names, never types.
- **Multiple inheritance / MRO**: a subclass may list several base classes; for a diamond, Python resolves names left to right, then upwards (C3 Method Resolution Ordering; inspect with `Cls.mro()`). There is no "correct" ordering — a language must just pick one consistently so programmers can predict behavior.
- **Separation of concerns**: each object encapsulates and manages part of the program's state; each class statement defines functions implementing part of the overall logic. OOP suits systems of separate interacting parts (users in a social network, characters in a game, shapes in a simulation). But functional abstractions remain the more natural metaphor for input-output relationships — don't force every bit of logic into a class.

## Evaluation Rules
The author states these procedures formally; memorize them — they resolve nearly all attribute confusion.

**Executing a class statement.** When `class <name>: <suite>` is executed: a new class is created and bound to `<name>` in the first frame of the current environment; the suite is then executed; any names bound within the suite, through `def` or assignment statements, create or modify attributes *of the class* (not of instances).

**Instantiation.** Calling a class (`Account('Kirk')`) creates a new object that is an instance of the class, then calls the constructor `__init__` with two arguments: the newly created object (bound to `self`, by near-universal convention) and the argument passed in the call. Only instantiation with call-expression syntax creates new user-defined objects; assignment to a new name merely aliases (`c = a; c is a` → `True`), and every instance has unique identity (`a is not b` even for identical constructor calls, compared with `is` / `is not`).

**Method definitions.** A `def` inside a class suite is declared like any function, but its effect differs: the function value is bound to the declared name *locally within the class as an attribute*. Every method's special first parameter `self` is bound to the object on which the method is invoked, giving all methods access to the object's state.

**Evaluating a dot expression** `<expression>.<name>` (the `<expression>` may be any expression; `<name>` must be a simple name):
1. Evaluate the `<expression>` to the left of the dot, yielding the *object* of the dot expression.
2. `<name>` is matched against the instance attributes of that object; if an attribute with that name exists, its value is returned.
3. If `<name>` does not appear among instance attributes, it is looked up in the class, yielding a class attribute value.
4. That value is returned **unless it is a function**, in which case a **bound method** is returned instead.

Instance attributes are found before class attributes, just as local names have priority over global in an environment. The bound method is formed in step 4 by combining the function with the object of the dot expression.

**Dual role of the object.** When a method is invoked via dot notation, the object plays two roles: it determines what the name means (`withdraw` is not a name in the environment but a name local to the class), *and* it is bound to the first parameter `self` when the method is invoked.

**Looking up a name in a class** (recursive; this is where inheritance enters):
1. If it names an attribute in the class, return the attribute value.
2. Otherwise, look up the name in the base class, if there is one.

The class of an object stays constant throughout: even when `deposit` is found in `Account`, it is called with `self` bound to the `CheckingAccount` instance, so subsequent `self.` lookups start from the subclass.

**Attribute assignment.** All assignment statements with a dot expression on the left-hand side affect attributes for the object of that dot expression. If the object is an instance, assignment sets an *instance* attribute; if the object is a class, assignment sets a *class* attribute. Consequence: assignment to an attribute of an object can never affect the attributes of its class.

**getattr/hasattr.** `getattr(obj, 'name')` behaves exactly like dot notation (string-keyed, like a dispatch dictionary): with an instance and a method name it returns a bound method; with a class as first argument it returns the attribute directly — a plain function. `hasattr(obj, 'name')` tests for a named attribute.

## Key Concepts
- **Class / instance**: a class is a template for all objects of that type; every object is an instance of some class; **instantiating** = creating a new instance, with function-call syntax.
- **Attribute**: a name-value pair associated with an object, accessible via dot notation.
- **Instance attribute**: attribute specific to a particular object (also called fields, properties, or instance variables); typically assigned in the constructor.
- **Class attribute**: shared across all objects of the class, created by assignment in the class suite outside any method (also called class variables or static variables).
- **Method**: function defined in a class suite that operates on the object; **invoked** on a particular object, which is passed implicitly as `self`.
- **Constructor**: the `__init__` method, called automatically on instantiation.
- **Bound method**: a value coupling a function with the object on which it will be invoked; `self` is pre-bound.
- **Dot expression**: `<expression>.<name>` — Python's syntactic formalization of message passing.
- **Base class / subclass**: base = parent = superclass; subclass = child class. Subclass inherits and may override.
- **Object interface**: a collection of attribute names and conditions on them; implemented implicitly in Python by having the names.
- **Inheritance graph / MRO**: arrows from subclass to base class; ambiguity in a diamond resolved left-to-right then upwards (C3), queryable via `mro()`.
- **Naming conventions**: CapWords (CamelCase) for classes; lowercase_with_underscores for methods; a leading underscore marks an attribute as implementation detail — accessed only within the class's own methods, not part of the abstraction.

## Mental Models
- A class is a dispatch dictionary with syntax: messages are names local to the class instead of string keys, state is instance attributes, and the language handles get/set through plain assignment — no `nonlocal` needed and no separate "get"/"set" messages.
- When tracing any attribute access, run the four-step dot-expression procedure explicitly, with the two-step recursive class lookup inside step 3.
- Instance namespace sits "in front of" the class namespace, like a local frame in front of the global frame.
- `self.x` in a method is late-bound: it re-runs the full lookup at call time, which is exactly why ancestors should reference shared values through `self`.
- Ask "is-a or has-a?" before inheriting; has-a means composition via instance attributes.

## Anti-patterns
- **Depending on object types instead of interfaces**: `Account.deposit(account, amount)` inside `deposit_all` violates the abstraction barrier and won't necessarily work with new kinds of accounts; `account.deposit(amount)` works with any object implementing the interface.
- **Hard-coding your own class inside methods**: `CheckingAccount.withdraw_charge` defeats overriding in further subclasses; use `self.withdraw_charge`.
- **Inheriting for has-a relationships**: a bank has accounts; neither bank nor account should inherit from the other.
- **Forcing all logic into classes**: independent functions manipulating data are often more natural and also enforce separation of concerns; choosing class vs function is an important design skill.

## Worked Examples

### 1. Account: constructor, methods, identity
```python
>>> class Account:
        def __init__(self, account_holder):
            self.balance = 0
            self.holder = account_holder
        def deposit(self, amount):
            self.balance = self.balance + amount
            return self.balance
        def withdraw(self, amount):
            if amount > self.balance:
                return 'Insufficient funds'
            self.balance = self.balance - amount
            return self.balance
```
`a = Account('Kirk')` creates the instance and runs `__init__` with `self` bound to it: `balance` starts at 0, `holder` persists as an attribute of `self` while the formal parameter `account_holder` is just a local name in `__init__`. Then:
```python
>>> spock_account = Account('Spock')
>>> spock_account.deposit(100)
100
>>> spock_account.withdraw(90)
10
>>> spock_account.withdraw(90)
'Insufficient funds'
```
Two calls to `withdraw` with the same argument return different results — method behavior depends on the object's changing attributes. Identity: `b = Account('Spock')` is a distinct object (`a is not b`), and `c = a` aliases (`c is a` is `True`). Function/method duality:
```python
>>> type(Account.deposit)
<class 'function'>
>>> type(spock_account.deposit)
<class 'method'>
>>> Account.deposit(spock_account, 1001)  # The deposit function takes 2 arguments
1011
>>> spock_account.deposit(1000)           # The deposit method takes 1 argument
2011
```

### 2. Class attributes and shadowing
With `interest = 0.02` in the class suite, both `spock_account.interest` and `kirk_account.interest` return `0.02` (step 3: not an instance attribute, found in the class). `Account.interest = 0.04` changes it for *all* instances in one assignment. Then:
```python
>>> kirk_account.interest = 0.08   # creates a NEW instance attribute, same name
>>> kirk_account.interest
0.08
>>> spock_account.interest         # class attribute unchanged
0.04
>>> Account.interest = 0.05        # changing the class attribute
>>> spock_account.interest         # changes instances without like-named instance attributes
0.05
>>> kirk_account.interest          # but the existing instance attribute is unaffected
0.08
```
Trace each line with the dot-expression procedure: Kirk's instance attribute wins at step 2; Spock falls through to step 3.

### 3. CheckingAccount: inheritance, override, calling the ancestor
```python
>>> class Account:
        """A bank account that has a non-negative balance."""
        interest = 0.02
        def __init__(self, account_holder):
            self.balance = 0
            self.holder = account_holder
        def deposit(self, amount):
            """Increase the account balance by amount and return the new balance."""
            self.balance = self.balance + amount
            return self.balance
        def withdraw(self, amount):
            """Decrease the account balance by amount and return the new balance."""
            if amount > self.balance:
                return 'Insufficient funds'
            self.balance = self.balance - amount
            return self.balance

>>> class CheckingAccount(Account):
        """A bank account that charges for withdrawals."""
        withdraw_charge = 1
        interest = 0.01
        def withdraw(self, amount):
            return Account.withdraw(self, amount + self.withdraw_charge)
```
```python
>>> checking = CheckingAccount('Sam')
>>> checking.deposit(10)
10
>>> checking.withdraw(5)
4
>>> checking.interest
0.01
```
Trace `checking.deposit(10)`: not an instance attribute; not in `CheckingAccount`; found in `Account` via recursive class lookup; it's a function, so a bound method is returned, then invoked with `self` = `checking`, `amount` = 10. `checking.withdraw(5)` finds the override in `CheckingAccount`, which calls the ancestor through the class object with explicit `self`, adding `self.withdraw_charge` (looked up through `self` so further subclasses can override the charge). Interface-respecting client code:
```python
>>> def deposit_all(winners, amount=5):
        for account in winners:
            account.deposit(amount)
```

### 4. Multiple inheritance: AsSeenOnTVAccount
```python
>>> class SavingsAccount(Account):
        deposit_charge = 2
        def deposit(self, amount):
            return Account.deposit(self, amount - self.deposit_charge)

>>> class AsSeenOnTVAccount(CheckingAccount, SavingsAccount):
        def __init__(self, account_holder):
            self.holder = account_holder
            self.balance = 1           # A free dollar!
```
This implementation is complete — withdrawal and deposit fees both apply, from `CheckingAccount` and `SavingsAccount` respectively:
```python
>>> such_a_deal = AsSeenOnTVAccount("John")
>>> such_a_deal.balance
1
>>> such_a_deal.deposit(20)            # $2 fee from SavingsAccount.deposit
19
>>> such_a_deal.withdraw(5)            # $1 fee from CheckingAccount.withdraw
13
>>> such_a_deal.deposit_charge         # non-ambiguous references resolve directly
2
>>> such_a_deal.withdraw_charge
1
```
The ambiguous name `withdraw` (defined in both `Account` and `CheckingAccount`) follows the diamond rule — left to right, then upwards:
```python
>>> [c.__name__ for c in AsSeenOnTVAccount.mro()]
['AsSeenOnTVAccount', 'CheckingAccount', 'SavingsAccount', 'Account', 'object']
```

## Reference Tables

Lookup order for the diamond `AsSeenOnTVAccount(CheckingAccount, SavingsAccount)`:

| Step | Class checked |
|---|---|
| 1 | AsSeenOnTVAccount |
| 2 | CheckingAccount |
| 3 | SavingsAccount |
| 4 | Account |
| 5 | object |

Function vs bound method:

| Access | Value type | Arity | `self` |
|---|---|---|---|
| `Account.deposit` | `function` | 2 (`self`, `amount`) | passed explicitly |
| `spock_account.deposit` | `method` | 1 (`amount`) | pre-bound to the instance |
| `getattr(spock_account, 'deposit')` | `method` | 1 | pre-bound (same as dot) |
| `getattr(Account, 'deposit')` | `function` | 2 | explicit |

Attribute assignment target:

| Left-hand side | Effect |
|---|---|
| `instance.name = v` | sets an instance attribute (may shadow a class attribute) |
| `Class.name = v` | sets the class attribute, seen by all instances without a like-named instance attribute |

## Common Errors
- **Confusing class and instance attributes**: expecting `kirk_account.interest = 0.08` to change the rate for everyone. It creates a *new instance attribute* with the same name; the class attribute retains its value and all other accounts are unaffected. Conversely, forgetting that `Account.interest = 0.05` silently changes every instance that hasn't shadowed it.
- **Shadowing then being surprised by class assignment**: once an instance attribute exists, later class-attribute changes never reach that instance (Kirk stays at 0.08 while Spock moves to 0.05).
- **Calling a function attribute as if it were bound**: `Account.deposit(1000)` fails — accessed through the class it is a plain function and needs `self` supplied explicitly: `Account.deposit(spock_account, 1001)`.
- **Expecting attribute assignment to reach the class**: by the assignment rule, assigning to an instance's attribute *cannot* affect its class's attributes.
- **Hard-coding the defining class for shared values**: `CheckingAccount.withdraw_charge` inside `withdraw` means a subclass's overridden charge is ignored; `self.withdraw_charge` finds the new value.
- **Client code calling through a concrete class**: `Account.deposit(account, amount)` in `deposit_all` breaks the abstraction barrier and fails for account-like objects that don't inherit from `Account`.
- **Treating locals in `__init__` as attributes**: `account_holder` is just a local name of the method; only `self.holder = account_holder` persists.
- **Assuming the found-in class changes `self`**: the class of an object stays constant; finding `deposit` in `Account` doesn't make `self` an `Account`.

## Key Takeaways
1. Instantiation is a call: new object created, then `__init__` runs with it bound to `self`; each instance has unique identity, and assignment aliases rather than copies.
2. Memorize the four-step dot-expression procedure (instance attributes, then class lookup, functions become bound methods) and the two-step recursive class lookup.
3. Class attributes are shared state; instance assignment shadows, class assignment broadcasts; the assignment target (instance vs class) decides which namespace changes.
4. Inherit for is-a, compose for has-a; override only what differs, and call ancestors through the class object with explicit `self` (`Account.withdraw(self, ...)`).
5. Reference shared values through `self.` inside methods so further subclasses can override them.
6. Program to interfaces — attribute names and conditions on them — never to types; that keeps the object abstraction's barrier intact (`account.deposit(amount)`, not `Account.deposit(account, amount)`).
7. Multiple-inheritance ambiguity resolves left-to-right, then upwards (C3); check `Cls.mro()` when in doubt; non-ambiguous names resolve directly.
8. Not everything belongs in a class: functional abstractions better express input-output relationships, and knowing when to add a class vs a function is a core design skill.

## Connects To
- **Section 2.4**: classes and dot notation formalize message passing and dispatch dictionaries with language syntax, replacing string-keyed messages and `nonlocal` state.
- **Section 2.6**: this whole object system — instances, classes, bound methods, inheritance — is implemented from scratch using only functions and dictionaries.
- **Section 2.7**: object abstraction develops interfaces further (string conversion, special methods, multiple representations, generic functions).
- **Chapter 1**: `self.` late binding is the environment model's local-before-global shadowing, replayed in the attribute namespace.
