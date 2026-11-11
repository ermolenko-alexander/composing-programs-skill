# Section 2.5: Object-Oriented Programming

## Core Idea
OOP unifies the chapter's techniques: like data abstraction, classes create abstraction barriers; like dispatch dictionaries, objects respond to behavioral requests; like mutable data, objects have local state. Programs become independent agents that interact, share behavior within a type, and inherit from related types.

## Frameworks Introduced
- **Class statement / constructor**: `class <name>:` binds a new class; names bound in its suite (via `def` or assignment) become class attributes. `__init__(self, ...)` is the **constructor**; instantiating uses call syntax (`Account('Kirk')`), creates the object, and calls `__init__` with it bound to `self`.
- **Dot expression evaluation procedure** (`<expression>.<name>`):
  1. Evaluate `<expression>` to get the object.
  2. Match `<name>` against the object's instance attributes; return if found.
  3. Otherwise look the name up in the class (yields a class attribute).
  4. Return that value — unless it is a function, in which case return a **bound method**.
  Instance attributes shadow class attributes, as local names shadow global.
- **Functions vs bound methods**: `Account.deposit` is a plain function (needs explicit `self`); `spock_account.deposit` is a bound method already coupled to its instance — same function body, different value type. `getattr`/`hasattr` are the functional equivalents of dot notation.
- **Class attributes vs instance attributes**: class attributes (assigned in the class suite, outside methods) are shared by all instances; assignment through an instance (`kirk_account.interest = 0.08`) creates a shadowing *instance* attribute and never changes the class attribute; assignment through the class changes all instances without a like-named instance attribute.
- **Inheritance**: `class CheckingAccount(Account):` — subclass inherits base-class attributes and may **override** them; specify only what differs. Class lookup is recursive: find the name in the class, else in the base class. Inheritance represents **is-a** relationships; **has-a** relationships should be instance attributes (a bank *has* accounts, it is not one).
- **Calling ancestors**: overridden attributes remain accessible via the class object (`Account.withdraw(self, amount + self.withdraw_charge)`); prefer `self.withdraw_charge` over `CheckingAccount.withdraw_charge` so further subclasses can override it.
- **Object interface**: a collection of attribute names and conditions on them; in Python any object with the appropriate names implements an interface (no declaration). Code using objects is most robust when it assumes only attribute names, never types.
- **Multiple inheritance / MRO**: a subclass may inherit from several bases; for a diamond, Python resolves names left to right, then upwards (C3 Method Resolution Ordering; inspect with `Cls.mro()`).
- **Separation of concerns**: each object encapsulates part of the program's state; each class defines part of the logic. But not everything belongs in a class — functional abstractions better express input-output relationships; choosing class vs function is a key design skill.

## Key Concepts
- **Class / instance**: a class is a template; every object is an instance of some class; instantiation = creating a new instance.
- **Instance attribute**: name-value pair specific to one object (also called fields, properties, instance variables).
- **Method**: function defined in a class suite; invoked on an object, which is passed implicitly as `self`.
- **Dual role of the dot-expression object**: it determines what the method name means (lookup in its class) *and* is bound to `self`.
- **Identity**: each instance of a user-defined class has unique identity; `a is not b` even for identical constructor calls; assignment aliases, never copies.
- **Base/subclass synonyms**: base = parent = superclass; subclass = child class.
- **Naming conventions**: CapWords (CamelCase) for classes; lowercase_with_underscores for methods; leading underscore = implementation detail, not part of the abstraction.
- **Class attribute synonyms**: class variables, static variables.

## Mental Models
- Think of a class as a dispatch dictionary with syntax: messages are class-local names, state is instance attributes, and the language handles get/set without nonlocal.
- When tracing a dot expression, run the four-step procedure explicitly — most attribute confusion dissolves.
- The class of an object stays constant: even when `deposit` is found in `Account`, `self` is still the `CheckingAccount` instance, so later `self.` lookups use the subclass.
- Ask "is-a or has-a?" before inheriting; use composition (instance attributes) for has-a.

## Anti-patterns
- **Depending on object types instead of interfaces**: `Account.deposit(account, amount)` in `deposit_all` breaks for new account classes; `account.deposit(amount)` works for anything implementing the interface.
- **Hard-coding your own class in methods**: `CheckingAccount.withdraw_charge` defeats overriding in further subclasses; use `self.`.
- **Expecting instance assignment to change the class attribute**: it silently creates a shadowing instance attribute.
- **Forcing all logic into classes**: independent functions are often the more natural abstraction.

## Code Examples
```python
class Account:
    """A bank account that has a non-negative balance."""
    interest = 0.02                      # A class attribute
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

class CheckingAccount(Account):
    """A bank account that charges for withdrawals."""
    withdraw_charge = 1
    interest = 0.01
    def withdraw(self, amount):
        return Account.withdraw(self, amount + self.withdraw_charge)
```
- **What it demonstrates**: the canonical Account/CheckingAccount pair — class attributes, override, and calling an ancestor through the class object.

## Reference Tables
MRO for the diamond `AsSeenOnTVAccount(CheckingAccount, SavingsAccount)`:

| Lookup order |
|---|
| AsSeenOnTVAccount → CheckingAccount → SavingsAccount → Account → object |

## Worked Example
The account hierarchy. `Account` holds `interest = 0.02`, `deposit`, `withdraw`. `CheckingAccount` overrides `interest = 0.01` and `withdraw` (adds `withdraw_charge = 1` by delegating to `Account.withdraw`). `SavingsAccount` adds `deposit_charge = 2` in an overridden `deposit`. `AsSeenOnTVAccount(CheckingAccount, SavingsAccount)` combines both fee behaviors with only an `__init__` giving a free dollar: `deposit(20)` yields 19 (savings fee), `withdraw(5)` yields 13 (checking fee). Non-ambiguous attributes resolve directly; the ambiguous `withdraw` follows the MRO. The interest experiments show shadowing: `kirk_account.interest = 0.08` pins Kirk at 0.08 while `Account.interest = 0.05` moves Spock to 0.05.

## Key Takeaways
1. Instantiation is a call; `__init__` sets instance attributes on `self`; each instance has unique identity.
2. Memorize the four-step dot-expression procedure: instance attrs, then class, functions become bound methods.
3. Class attributes are shared state; instance assignment shadows, class assignment broadcasts.
4. Inherit for is-a, compose for has-a; override only what differs and delegate upward via the base class object with explicit `self`.
5. Reference shared state through `self.` so subclasses can override it.
6. Program to interfaces (attribute names), not types — that keeps abstraction barriers intact.
7. Multiple-inheritance ambiguity resolves left-to-right, then upwards; check `mro()` when in doubt.
8. Don't put everything in classes; match the paradigm to the problem.

## Connects To
- **Section 2.4**: classes formalize message passing and dispatch dictionaries with language syntax.
- **Section 2.6**: this whole object system is implemented from scratch with functions and dictionaries.
- **Section 2.7**: interfaces and generic functions extend "program to attribute names" into object abstraction.
