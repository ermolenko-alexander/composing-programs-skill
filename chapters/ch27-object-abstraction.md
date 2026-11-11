# Section 2.7: Object Abstraction

## Core Idea
The object system lets multiple representations of abstract data coexist in one program. The central concept is the **generic function** — a function accepting values of multiple types — implemented by three techniques: shared interfaces, type dispatching, and type coercion.

## Frameworks Introduced
- **Two string representations**: every object should produce (1) `str` — human-interpretable text, and (2) `repr` — a Python expression such that, for most types, `eval(repr(object)) == object` (the "canonical string representation"); interactive sessions print `repr`. Both are polymorphic: `repr` invokes `__repr__`, `str` invokes `__str__` — so implementing those methods extends existing built-in functions to new classes.
- **Special methods** (special names invoked by the interpreter): `__init__` on construction; `__repr__`/`__str__` for display; `__bool__` for truth value (default: user-defined objects are true; sequences fall back to `__len__`, empty = false); `__len__` for `len`; `__getitem__` for element selection; `__call__` makes instances callable like functions; `__add__`/`__radd__` and friends define operator behavior (Python checks `__add__` on the left operand, then `__radd__` on the right).
- **Interface** (shared-interface technique): a set of shared attribute names plus a specification of their behavior. Complex arithmetic needs exactly `real`, `imag`, `magnitude`, `angle` — any class providing them, consistently, works. The interface approach is **additive**: a third representation needs only another class with the same attributes.
- **@property**: compute attributes on the fly from zero-argument methods, accessed without call syntax — the solution when two attribute sets must maintain a fixed relationship (store one representation, derive the other).
- **Type dispatching**: inspect argument types and execute type-appropriate code — via `isinstance(obj, Cls)` (true if the object's class is or inherits from Cls) or via a `type_tag` string attribute (showing dispatching is a general technique, not tied to the object system). Cross-type operations live in dictionaries (`adders`, `multipliers`) keyed by `(type_tag, type_tag)` pairs; extensible by installing new entries.
- **Type coercion**: view one type as another by converting with a coercion function (`rational_to_complex`); the `coercions` dictionary maps `(from_tag, to_tag)` to converters; `coerce` returns two same-typed values, then ordinary same-type `add`/`mul` applies. One function per pair of types replaces one function per pair per operation — because the right transformation depends only on the types, not the operation. Extensions: coerce both types to a common third type (rhombus and rectangle → quadrilateral); iterative coercion through intermediate types.

## Key Concepts
- **Generic/polymorphic function**: applicable to many ("poly") forms ("morph") of data; `Complex.add` is generic over `ComplexRI`/`ComplexMA`.
- **Number superclass**: defines `__add__`/`__mul__` in terms of abstract `add`/`mul`; not meant to be instantiated — subclasses supply the methods.
- **ComplexRI / ComplexMA**: rectangular (stores real/imag, derives magnitude/angle via `atan2` and Pythagoras) vs polar (stores magnitude/angle, derives real/imag via cos/sin).
- **Behavior condition for the interface**: `(real, imag)` and `(magnitude, angle)` must describe the same point on the complex plane.
- **type_tag**: class attribute string identifying the mathematical type (`'rat'`, `'com'`) shared by all representations of it.
- **cross_apply**: looks up the cross-type function for `(self.type_tag, other.type_tag)` and applies it; commutative ops reuse one implementation with swapped arguments.
- **Coercion loss**: coercion can lose information — exact rationals become float approximations as complex numbers.

## Mental Models
- Think of special methods as the hooks by which built-in functions and operators become generic: define `__x__` and your class joins the protocol.
- Choose the technique by structure: shared interface when representations of the *same* abstraction coexist; type dispatching when unrelated types need explicit cross-operations; coercion when one type can be *viewed as* another.
- Count functions to compare dispatching vs coercion: dispatching needs n functions per operation per type pair; coercion needs one per type pair total.
- `@property` = stored vs derived attributes: pick the representation you store; derive the rest so consistency is automatic.

## Anti-patterns
- **Writing cross-type functions for every operation** when a single coercion would do.
- **Coercing when exactness matters**: rational → complex discards exact representation.
- **Automatic language-level coercion**: Python removed `__coerce__` because its complexity did not justify its use; operators coerce explicitly as needed.
- **Instantiating abstract superclasses** like `Number`, which exists only to define the operator protocol.

## Code Examples
```python
class Number:
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
- **What it demonstrates**: the coercion-based Number class — one coercion entry replaces four cross-type functions.

## Reference Tables
| Technique | Mechanism | Cost of adding a type |
|---|---|---|
| Shared interface | same attribute names + behavior conditions | one new class (additive) |
| Type dispatching | type_tag / isinstance + cross-op dictionaries | one cross-function per op per type pair |
| Type coercion | coercions dict of converters | one converter per type pair |

## Worked Example
Mixed-type arithmetic. `Number` defines `__add__`/`__mul__`. `Complex(Number)` implements `add` in rectangular terms (returns `ComplexRI(self.real + other.real, self.imag + other.imag)`) and `mul` in polar terms (returns `ComplexMA(product of magnitudes, sum of angles)`). `ComplexRI` stores real/imag with `@property magnitude/angle`; `ComplexMA` the reverse — so either representation works in either role: `ComplexRI(1, 2) + ComplexMA(2, pi/2)` → `ComplexRI(1, 4)`; `ComplexRI(0, 1) * ComplexRI(0, 1)` → `ComplexMA(1, 1 * pi)`. Adding `Rational(Number)` (gcd-reduced numer/denom, `add`/`mul` as in 2.2) raises the cross-type problem: first solved by type dispatching (`adders`/`multipliers` keyed by tag pairs, with `add_complex_and_rational` etc.), then more economically by coercion (`rational_to_complex(r) = ComplexRI(r.numer/r.denom, 0)`), after which `ComplexRI(1.5, 0) + Rational(3, 2)` → `ComplexRI(3, 0)`.

## Key Takeaways
1. Always give classes `__repr__` (expression that reconstructs the value) and `__str__` (human text); interactive display uses repr.
2. Special methods are the extension points of Python: `__bool__`, `__len__`, `__getitem__`, `__call__`, `__add__`/`__radd__` make your objects participate in built-in protocols.
3. Define generic operations against an interface — a set of attribute names plus behavior conditions — and new representations become additive.
4. Use `@property` to keep derived attributes consistent with stored ones.
5. Prefer coercion over per-operation cross-type dispatch when types nest naturally; beware information loss.
6. Type dispatching via tags works in any language — it is a technique, not a Python feature.

## Connects To
- **Section 2.2**: multiple representations generalize data abstraction — the same "names + behavior conditions" idea.
- **Section 2.5**: builds directly on interfaces and inheritance (Number as abstract base).
- **Section 2.9**: `__repr__`, `__len__`, `__getitem__`, `__contains__` reappear in Link/Tree/sets.
