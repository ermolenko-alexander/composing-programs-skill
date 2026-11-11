# Section 2.7: Object Abstraction

> Source: "Composing Programs" by John DeNero, §2.7 (CC BY-SA 3.0). Code listings reproduced from the text; prose synthesized.

## Core Idea
The object system lets programmers build and use abstract data representations efficiently — and, crucially, lets **multiple representations of abstract data coexist in the same program**. The central concept is the **generic function**: a function that can accept values of multiple different types. Three techniques implement generic functions: **shared interfaces**, **type dispatching**, and **type coercion**. Along the way, the Python object system reveals the features that support generic functions — above all, special method names invoked by the interpreter.

Why multiple representations matter: large software systems are built by many people over long periods under changing requirements; agreeing in advance on one data representation is simply not possible. Beyond the data-abstraction barriers that isolate representation from use, we need abstraction barriers that isolate different design choices from each other and permit them to coexist in a single program.

## Frameworks Introduced

- **Two string representations**: Python stipulates every object should produce two strings — one human-interpretable (`str`) and one Python-interpretable (`repr`). The `repr` docstring: "Return the canonical string representation of the object. For most object types, eval(repr(object)) == object." Interactive sessions print the `repr` of each expression's value. Where no expression evaluates back to the value, Python produces an angle-bracket description, e.g. `repr(min)` → `'<built-in function min>'`.
- **Special methods** ("special names invoked by the Python interpreter in special circumstances"): `__init__` on construction; `__repr__`/`__str__` for display; `__bool__` for truth value; `__len__` for `len`; `__getitem__` for element selection; `__call__` to make instances callable; `__add__`/`__radd__` and the rest of the operator protocol.
- **Interface**: a set of shared attribute names, along with a specification of their behavior. Object attributes are a form of message passing; a shared set of messages eliciting similar behavior from different classes is a powerful abstraction method. For complex arithmetic the interface is exactly four attributes: `real`, `imag`, `magnitude`, `angle`.
- **@property**: computes attributes on the fly from zero-argument functions, accessed without call-expression syntax. The solution when two attribute sets must maintain a fixed relationship: store one representation, compute the other on demand.
- **Type dispatching**: write functions that inspect the types of their arguments, then execute type-appropriate code — via `isinstance(obj, Cls)` or via a `type_tag` string attribute. Cross-type operations live in dictionaries (`adders`, `multipliers`) keyed by pairs of type tags.
- **Type coercion**: view objects of one type as being of another by converting them with a coercion function; the `coercions` dictionary maps `(from_tag, to_tag)` pairs to converters. After coercion, ordinary same-type `add`/`mul` apply. Extensions: coerce two different types each into a third common type (a rhombus and a rectangle are both quadrilaterals); iterative coercion through intermediate types (integer → rational → real), which reduces the total number of coercion functions a program needs.

## Evaluation Rules (author's formal procedures)

1. **`repr` invocation**: `repr(x)` always invokes the method `__repr__` on its argument — `x.__repr__()`. Implementing `__repr__` in a user-defined class extends the applicability of `repr` to any class created in the future. Dot expressions in general provide this mechanism for extending the domain of existing functions to new object types.
2. **`str` invocation**: the `str` constructor is implemented in the same manner — it invokes `__str__` on its argument. `str` often coincides with `repr`, but gives more interpretable text in some cases (e.g. `repr(date(2011, 9, 12))` → `'datetime.date(2011, 9, 12)'` vs `str` → `'2011-09-12'`).
3. **Truth value**: all Python objects have a truth value. By default, instances of user-defined classes are true; if the class defines `__bool__`, Python calls it to determine truth. If no `__bool__` exists, Python falls back on a sequence's **length**: empty sequences are false, non-empty ones true (`bool('')` → `False`, `bool([])` → `False`).
4. **`len` and element selection**: `len(s)` invokes `s.__len__()`; the element-selection operator `s[i]` invokes `s.__getitem__(i)` — both can also be invoked directly (`'Go Bears!'.__getitem__(3)` → `'B'`).
5. **Callable objects**: defining `__call__` lets instances be called like functions, blurring the line between data and functions further.
6. **Operator protocol for `+`**: Python follows a specific protocol per operator. For `+`, Python first checks for an `__add__` method on the value of the **left** operand, then checks for an `__radd__` method on the value of the **right** operand. If either is found, that method is invoked with the value of the **other** operand as its argument. (The Python docs list the exhaustive set of operator method names.)
7. **`isinstance` rule**: `isinstance(obj, Cls)` returns true if the object has a class that either **is or inherits from** the given class — `isinstance(ComplexRI(1,1), Complex)` is `True`, `isinstance(ComplexRI(1,1), ComplexMA)` is `False`.
8. **Dispatching `__add__` (type-tag version)**: two cases. If `self.type_tag == other.type_tag`, assume `self.add(other)` applies. Otherwise, if `(self.type_tag, other.type_tag)` is a key in the `adders` dictionary, `cross_apply` looks up the cross function by that tag pair and applies it to `(self, other)`. `__mul__` is identical over `multipliers`.
9. **Coercing `coerce` method**: returns two values with the same type tag. If the tags already match, return `(self, other)` unchanged. Else if `(self.type_tag, other.type_tag)` is in `coercions`, return `(self.coerce_to(other.type_tag), other)`. Else if the reversed pair is in `coercions`, return `(self, other.coerce_to(self.type_tag))`. `coerce_to` looks up `coercions[(self.type_tag, other_tag)]` and applies that function to `self`.
10. **Why one coercion function suffices per type pair**: the appropriate transformation between types depends **only on the types themselves, not on the particular operation** to be applied — so one converter replaces one cross function per operation.

## Key Concepts
- **Generic/polymorphic function**: applicable to many ("poly") different forms ("morph") of data. `repr` itself is the motivating example — it must apply correctly even to types that did not exist when it was implemented.
- **Canonical string representation**: `repr`'s output; `eval(repr(object)) == object` for most types.
- **Special names**: method names the interpreter invokes automatically (`__init__`, `__repr__`, `__str__`, `__bool__`, `__len__`, `__getitem__`, `__call__`, `__add__`, `__radd__`, `__mul__`).
- **Interface**: shared attribute names + behavior specification; implicitly defined by how `Complex` *uses* the four attributes in `add` and `mul`.
- **Behavior condition**: for correctness, `(real, imag)` and `(magnitude, angle)` must describe the same point on the complex plane.
- **Additive interface**: a third representation of complex numbers requires only another class with the same attributes — no existing code changes.
- **`Number` superclass**: requires `add`/`mul` but does not define them, and has no `__init__`; its purpose is not to be instantiated but to serve as a superclass of specific number classes.
- **`type_tag`**: class-attribute string identifying the mathematical type (`Rational.type_tag = 'rat'`, `Complex.type_tag = 'com'`) — shared by all representations of it, so `ComplexRI` and `ComplexMA` compare equal by tag.
- **`cross_apply`**: `cross_fn = cross_fns[(self.type_tag, other.type_tag)]; return cross_fn(self, other)`.
- **Coercion**: viewing an object of one type as being of another type via a conversion function.
- **Coercion loss**: coercion functions can lose information — rationals are exact, but become approximations when converted to complex.
- **Commutativity reuse**: addition and multiplication are commutative, so the swapped-argument cross functions simply delegate (`add_rational_and_complex(r, c)` returns `add_complex_and_rational(c, r)`).

## Mental Models
- Special methods are the hooks by which built-in functions and operators become generic: define `__x__` and your class joins the protocol. Dot expressions extend the domain of existing functions to new types.
- Choose the technique by structure: **shared interface** when multiple representations of the *same* abstraction must coexist; **type dispatching** when completely unrelated types need explicit, controlled cross-operations; **coercion** when latent structure lets one type be *viewed as* another.
- `@property` = stored vs derived attributes: pick the representation to store; derive the rest so the behavior condition holds automatically. Mutating a stored attribute (`ri.real = 9`) is instantly reflected in the derived ones (`ri.magnitude` → `15.0`).
- Count functions to compare: dispatching needs a cross function per operation per ordered type pair; coercion needs one converter per type pair, period.
- Type dispatching via tags is not linked to the Python object system — it is a general technique for creating generic functions over heterogeneous domains, usable in any language.
- There is a tension to manage: we want a generic `__add__` that does the right thing with all numeric types, while separating the concerns of complex and rational numbers to keep the program modular. Cross-type operations must be introduced "in some carefully controlled way."

## Code Examples

### Two string representations and special-method invocation
```python
>>> tues.__repr__()          # what repr(tues) invokes
'datetime.date(2011, 9, 12)'
>>> tues.__str__()           # what str(tues) invokes
'2011-09-12'
>>> Account.__bool__ = lambda self: self.balance != 0
>>> bool(Account('Jack'))
False
>>> 'Go Bears!'.__len__()
9
>>> 'Go Bears!'.__getitem__(3)
'B'
```

### Callable objects: `Adder` mirrors `make_adder`
```python
>>> def make_adder(n):
        def adder(k):
            return n + k
        return adder
>>> make_adder(3)(4)
7

>>> class Adder(object):
        def __init__(self, n):
            self.n = n
        def __call__(self, k):
            return self.n + k
>>> add_three_obj = Adder(3)
>>> add_three_obj(4)
7
```
`Adder` behaves like the higher-order function `make_adder`; `add_three_obj` behaves like the function `add_three`.

### Interface version: Number / Complex / ComplexRI / ComplexMA
```python
>>> class Number:
        def __add__(self, other):
            return self.add(other)
        def __mul__(self, other):
            return self.mul(other)

>>> class Complex(Number):
        def add(self, other):
            return ComplexRI(self.real + other.real, self.imag + other.imag)
        def mul(self, other):
            magnitude = self.magnitude * other.magnitude
            return ComplexMA(magnitude, self.angle + other.angle)

>>> from math import atan2
>>> class ComplexRI(Complex):
        def __init__(self, real, imag):
            self.real = real
            self.imag = imag
        @property
        def magnitude(self):
            return (self.real ** 2 + self.imag ** 2) ** 0.5
        @property
        def angle(self):
            return atan2(self.imag, self.real)
        def __repr__(self):
            return 'ComplexRI({0:g}, {1:g})'.format(self.real, self.imag)

>>> from math import sin, cos, pi
>>> class ComplexMA(Complex):
        def __init__(self, magnitude, angle):
            self.magnitude = magnitude
            self.angle = angle
        @property
        def real(self):
            return self.magnitude * cos(self.angle)
        @property
        def imag(self):
            return self.magnitude * sin(self.angle)
        def __repr__(self):
            return 'ComplexMA({0:g}, {1:g} * pi)'.format(self.magnitude, self.angle/pi)
```

### Rational with Number's interface
```python
>>> from fractions import gcd
>>> class Rational(Number):
        def __init__(self, numer, denom):
            g = gcd(numer, denom)
            self.numer = numer // g
            self.denom = denom // g
        def __repr__(self):
            return 'Rational({0}, {1})'.format(self.numer, self.denom)
        def add(self, other):
            nx, dx = self.numer, self.denom
            ny, dy = other.numer, other.denom
            return Rational(nx * dy + ny * dx, dx * dy)
        def mul(self, other):
            numer = self.numer * other.numer
            denom = self.denom * other.denom
            return Rational(numer, denom)

>>> Rational(2, 5) + Rational(1, 10)
Rational(1, 2)
>>> Rational(1, 4) * Rational(2, 3)
Rational(1, 6)
```

### Cross-type functions and the type-dispatching Number
```python
>>> def add_complex_and_rational(c, r):
        return ComplexRI(c.real + r.numer/r.denom, c.imag)

>>> def mul_complex_and_rational(c, r):
        r_magnitude, r_angle = r.numer/r.denom, 0
        if r_magnitude < 0:
            r_magnitude, r_angle = -r_magnitude, pi
        return ComplexMA(c.magnitude * r_magnitude, c.angle + r_angle)

>>> def add_rational_and_complex(r, c):
        return add_complex_and_rational(c, r)
>>> def mul_rational_and_complex(r, c):
        return mul_complex_and_rational(c, r)

>>> class Number:
        def __add__(self, other):
            if self.type_tag == other.type_tag:
                return self.add(other)
            elif (self.type_tag, other.type_tag) in self.adders:
                return self.cross_apply(other, self.adders)
        def __mul__(self, other):
            if self.type_tag == other.type_tag:
                return self.mul(other)
            elif (self.type_tag, other.type_tag) in self.multipliers:
                return self.cross_apply(other, self.multipliers)
        def cross_apply(self, other, cross_fns):
            cross_fn = cross_fns[(self.type_tag, other.type_tag)]
            return cross_fn(self, other)
        adders = {("com", "rat"): add_complex_and_rational,
                  ("rat", "com"): add_rational_and_complex}
        multipliers = {("com", "rat"): mul_complex_and_rational,
                       ("rat", "com"): mul_rational_and_complex}
```

### Coercion-based Number
```python
>>> def rational_to_complex(r):
        return ComplexRI(r.numer/r.denom, 0)

>>> class Number:
        def __add__(self, other):
            x, y = self.coerce(other)
            return x.add(y)
        def __mul__(self, other):
            x, y = self.coerce(other)
            return x.mul(y)
        def coerce(self, other):
            if self.type_tag == other.type_tag:
                return self, other
            elif (self.type_tag, other.type_tag) in self.coercions:
                return (self.coerce_to(other.type_tag), other)
            elif (other.type_tag, self.type_tag) in self.coercions:
                return (self, other.coerce_to(self.type_tag))
        def coerce_to(self, other_tag):
            coercion_fn = self.coercions[(self.type_tag, other_tag)]
            return coercion_fn(self)
        coercions = {('rat', 'com'): rational_to_complex}
```
One `coercions` entry replaces the four cross-type functions of the dispatching version.

## Reference Tables

### Techniques for generic functions
| Technique | Mechanism | Scope | Cost of extension |
|---|---|---|---|
| Shared interface | same attribute names + behavior conditions; message passing | representations of the *same* abstraction (ComplexRI/ComplexMA) | additive — one new class, developed separately; agreement only on shared names and behavior conditions |
| Type dispatching | `type_tag` (or `isinstance`) + `adders`/`multipliers` dicts keyed by tag pairs | unrelated types needing controlled cross-operations | one cross function per operation per ordered type pair; new subclasses install themselves by declaring a tag and adding entries |
| Type coercion | `coercions` dict of converter functions keyed by `(from, to)` tag pairs | types with latent structure — one viewable as another | one converter per type pair (operation-independent) |

### Cross-type operation table (type dispatching)
| Dict | Key | Value |
|---|---|---|
| `adders` | `("com", "rat")` | `add_complex_and_rational` |
| `adders` | `("rat", "com")` | `add_rational_and_complex` (swap + delegate) |
| `multipliers` | `("com", "rat")` | `mul_complex_and_rational` |
| `multipliers` | `("rat", "com")` | `mul_rational_and_complex` (swap + delegate) |
| `coercions` (coercion version) | `('rat', 'com')` | `rational_to_complex` — the only entry needed |

### Dispatching vs coercion trade-offs (as discussed in the text)
| Aspect | Type dispatching | Coercion |
|---|---|---|
| Functions needed | per operation per type pair | one per type pair — transformation depends only on types, not the operation |
| When it's the best option | "completely unrelated operations acting on completely unrelated types" — explicit cross-type ops, cumbersome though they may be, are the best one can hope for | when latent structure exists: one type viewable as another |
| Information | preserves both representations (works on both simultaneously) | can lose information (exact rational → approximate complex) |
| Extensions | subclasses add tags + dict entries, or define their own `adders`/`multipliers` | coerce two types to a common third type; iterative coercion via intermediate types |
| Limits | dictionary grows with ops × pairs | not every coercion exists — no way to coerce an arbitrary complex to a rational, so no such entry |

## Worked Examples

### 1. Multiple representations via a shared interface
`Number` defines `__add__`/`__mul__` as `self.add(other)`/`self.mul(other)`. `Complex(Number)` implements `add` in rectangular terms (adding `real` and `imag` coordinates — a complex number as a point in the plane) and `mul` in polar terms (the product stretches one vector by the other's length and rotates it through the other's angle). `ComplexRI` stores `real`/`imag` and derives `magnitude` (Pythagoras) and `angle` (`atan2`) as properties; `ComplexMA` stores `magnitude`/`angle` and derives `real`/`imag` via `cos`/`sin`. Because all four attributes are accessed without call expressions, either class works for either operand of either operation:
```python
>>> ComplexRI(1, 2) + ComplexMA(2, pi/2)
ComplexRI(1, 4)
>>> ComplexRI(0, 1) * ComplexRI(0, 1)
ComplexMA(1, 1 * pi)
```
And properties keep the behavior condition automatic: `ri = ComplexRI(5, 12)` has `ri.magnitude == 13.0`; after `ri.real = 9`, `ri.magnitude == 15.0`. For `ma = ComplexMA(2, pi/2)`, `ma.imag == 2.0`; after `ma.angle = pi`, `ma.real == -2.0`.

### 2. Cross-type arithmetic by type dispatching
Add `Rational(Number)` (gcd-reduced). Now `Rational + Complex` is mathematically well-defined but unsupported. First assign tags (`Rational.type_tag = 'rat'`, `Complex.type_tag = 'com'`); representations of the same type share a tag (`ComplexRI(1,1).type_tag == ComplexMA(2,pi/2).type_tag` → `True`, while `'rat' == 'com'` → `False`). Write `add_complex_and_rational` (converts the rational to a float real part) and `mul_complex_and_rational` (a real number in polar form always has positive magnitude; angle 0 means positive, angle `pi` means negative — hence the sign flip). Commutativity gives the swapped versions for free. The rewritten `Number.__add__` dispatches: same tag → `self.add(other)`; else look up the tag pair in `adders` and `cross_apply`. Result:
```python
>>> ComplexRI(1.5, 0) + Rational(3, 2)
ComplexRI(3, 0)
>>> Rational(-1, 2) * ComplexMA(4, pi/2)
ComplexMA(2, 1.5 * pi)
```
A simpler dispatching example uses `isinstance`: `is_real(c)` returns `c.imag == 0` for a `ComplexRI` but `c.angle % pi == 0` for a `ComplexMA`.

### 3. The same system by coercion
Observe the latent structure: a rational can be *viewed as* a complex number with zero imaginary part. One converter, `rational_to_complex(r) = ComplexRI(r.numer/r.denom, 0)`, plus a `coerce` method that returns two same-tagged values (checking the pair and its reverse in `coercions`), lets `__add__` become simply `x, y = self.coerce(other); return x.add(y)`. The one dictionary entry `{('rat', 'com'): rational_to_complex}` replaces all four cross functions — because the right transformation depends only on the types, not the operation. The price: exactness is lost on conversion.

### 4. Special-methods example: operator and call protocols
`Rational(2, 5) + Rational(1, 10)` evaluates by the `+` protocol: Python finds `__add__` on the left operand (inherited from `Number`) and invokes it with the right operand. Truth and length follow the same pattern: after `Account.__bool__ = lambda self: self.balance != 0`, `if not Account('Jack'):` prints `'Jack has nothing'` because the interpreter calls `__bool__` in the boolean context. And `Adder(3)(4)` → `7` because the call expression on an instance invokes `__call__`.

## Anti-patterns
- **Writing cross-type functions for every operation** when one coercion per type pair would do.
- **Coercing when exactness matters**: rational → complex discards the exact representation.
- **Expecting impossible coercions**: there is no way to coerce an arbitrary complex number to a rational — don't fake one; just omit the entry.
- **Automatic language-level coercion**: early Python had a `__coerce__` special method; the complexity of the built-in coercion system did not justify its use, and it was removed. Instead, particular operators apply coercion to their arguments as needed.
- **Instantiating abstract superclasses** like `Number`, which has no `__init__` and exists only to be a superclass defining the operator protocol.

## Common Errors
- **Omitting `__repr__`/`__str__`**: your objects then can't participate in the generic `repr`/`str` that interactive Python relies on; where possible `repr` output should be an expression that evaluates to an equal object.
- **Breaking the behavior condition**: if `(real, imag)` and `(magnitude, angle)` describe different points, mixed-representation arithmetic silently produces wrong answers. Store one representation; derive the other with `@property`.
- **Accessing properties with call syntax**: `@property` attributes are read without parentheses — the whole point is that all four interface attributes are accessed uniformly, without call expressions.
- **Forgetting that representations of one type share a tag**: `type_tag` belongs to the mathematical type (`Complex`), not the representation class — set it on `Complex`, not per subclass.
- **Ignoring sign in polar conversion**: a polar magnitude must be positive; `mul_complex_and_rational` must flip a negative rational to `(-magnitude, pi)`.
- **Only handling one operand order**: both `("com","rat")` and `("rat","com")` entries are needed; for commutative ops the second just swaps arguments.
- **Assuming dispatching requires the object system**: type tags show it is a general technique over heterogeneous domains, independent of Python classes.
- **Losing information silently via coercion** — know that the converted value is an approximation.

## Key Takeaways
1. Give every class `__repr__` (a canonical, ideally eval-able expression) and `__str__` (human text); interactive display uses `repr`, and both are generic because they dispatch through `__repr__`/`__str__` on the argument.
2. Special methods are Python's extension points: `__bool__` (falling back to `__len__` for sequences), `__len__`, `__getitem__`, `__call__`, and the operator protocol (`__add__` on the left, then `__radd__` on the right) make your objects participate in built-in behavior.
3. Define generic operations against an interface — shared attribute names plus behavior conditions — and new representations become purely additive, developed separately.
4. Use `@property` to store one representation and compute the other, so consistency between attribute sets is automatic.
5. Type dispatching with tag-pair dictionaries supports controlled, extensible cross-type operations; new types install themselves by declaring a tag and adding entries.
6. Prefer coercion over per-operation dispatch when one type can be viewed as another: one converter per type pair, since the transformation depends on types, not operations. Beware information loss, and consider common-type and iterative coercion to shrink the converter set.
7. Both data abstraction and multiple representations rest on the same foundation: a set of names and corresponding behavior conditions.

## Connects To
- **Section 2.2 (Data Abstraction)**: `Rational.add`/`mul` express the same computations as `add_rational`/`mul_rational`; multiple representations generalize the chapter-opening idea — change representation without changing meaning.
- **Section 2.4 (Dispatch Dictionaries)**: the `adders`/`multipliers`/`coercions` dictionaries are dispatch dictionaries keyed by type instead of message.
- **Section 2.5 (OOP)**: builds on inheritance and message passing; `Number` as an abstract superclass; dot expressions extend existing functions to new types.
- **Section 2.9 (Recursive Objects)**: `__repr__`, `__len__`, `__getitem__` reappear in the `Link`, `Tree`, and set classes.
