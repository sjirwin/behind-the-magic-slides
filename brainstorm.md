# Descriptors

## What the descriptor protocol primarily controls

The descriptor protocol primarily controls attribute access.

Specifically, it allows objects to customize the behavior when they are accessed, set, or deleted as attributes of other classes.

The protocol is defined by these special methods:

- `__get__(self, instance, owner)` - controls attribute retrieval
- `__set__(self, instance, value)` - controls attribute assignment
- `__delete__(self, instance)` - controls attribute deletion

Common examples of descriptors include:

- Properties (`@property`)
- Methods (which are non-data descriptors)
- Class methods and static methods
- Slots

Descriptors that define `__get__` and `__set__` are called "data descriptors" and take priority over instance dictionary entries during attribute lookup. Those with only `__get__` are "non-data descriptors" and have lower priority.

## Does the descriptor protocol control anything else

No, the descriptor protocol primarily controls just attribute access (get, set, delete operations).

However, there are important consequences and use cases that stem from this control:

1. **Method binding** - When you access a method on an instance, the descriptor protocol (`__get__`) is what creates the bound method that automatically passes `self`. But this is still fundamentally attribute access.
2. **Attribute lookup order** - Data descriptors (those with `__get__` and `__set__`) take priority over instance `__dict__` entries, while non-data descriptors don't. This influences Python's attribute resolution, but again, it's part of e attribute access mechanism.

Any other behaviors you see with descriptors (validation, type checking, lazy loading, etc.) are applications or patterns built on top of the core attribute access control - they're not separate things the protocol controls.

So the straightforward answer is: the descriptor protocol controls attribute access, and everything else flows from that single purpose.

## The lookup order as it relates to the descriptor protocol and the instance `__dict__` when Python does attribute lookup

When Python does attribute lookup (e.g., `instance.attr`), the order is:

1. Data descriptors from the `type(instance)` and its bases
2. Instance `__dict__` (i.e., `instance.__dict__['attr']`)
3. Non-data descriptors from `type(instance)` and its bases, and other class attributes
4. Raises `AttributeError`

### Key Distinction

- **Data descriptor**: defines `__get__` and at least one of `__set__` or `__delete__`
- **Non-data descriptor**: defines only `__get__`

### Why this matters

Data descriptors (like `@property`) take priority over instance dictionary:

### Illustrative Example 1

```python
class Example:
    @property
    def x(self):
        return "property"

obj = Example()
obj.__dict__['x'] = "instance attr"
print(obj.x)  # "property" - data descriptor wins
```

Non-data descriptors (like methods) can be overridden by instance attributes:

```python
class Example:
    def method(self):
        return "method"

obj = Example()
obj.method = "instance attr"
print(obj.method)  # "instance attr" - instance dict wins
```

This lookup order is what makes properties work for validation/control while still allowing instance attributes to shadow methods when needed.

### Illustrative Example 2

```python
class DataDescriptor:
    """Has __get__ and __set__ - takes priority over instance __dict__"""
    def __get__(self, instance, owner):
        return "data descriptor"
    def __set__(self, instance, value):
        pass


class NonDataDescriptor:
    """Has only __get__ - lower priority than instance __dict__"""
    def __get__(self, instance, owner):
        return "non-data descriptor"


class Demo:
    data_desc = DataDescriptor()
    non_data_desc = NonDataDescriptor()
    class_attr = "class attribute"


# Create instance and add instance attributes
obj = Demo()
obj.__dict__['data_desc'] = "instance value 1"
obj.__dict__['non_data_desc'] = "instance value 2"
obj.__dict__['class_attr'] = "instance value 3"

# Data descriptor wins over instance __dict__
print(obj.data_desc)  # "data descriptor"

# Instance __dict__ wins over non-data descriptor
print(obj.non_data_desc)  # "instance value 2"

# Instance __dict__ wins over class attribute
print(obj.class_attr)  # "instance value 3"
```

Properties are data descriptors (they have `__set__` even if it just raises `AttributeError`), so they always override instance attributes:

```python
class Point:
    @property
    def x(self):
        return self._x

p = Point()
p.__dict__['x'] = 100  # Sneaky attempt to override
print(p.x)  # Still calls the property getter!

Methods are non-data descriptors, so you can override them per-instance:

class MyClass:
    def method(self):
        return "original"

obj = MyClass()
obj.method = lambda: "overridden"  # This works!
print(obj.method())  # "overridden"
```

## The lookup order if there are class level attributes

Class attributes are checked in the same step as non-data descriptors. Here's the complete lookup order:

Complete Attribute Lookup Order (for `obj.attr`)

1. Data descriptors from `type(obj).__mro__` (class and its bases)
2. Instance `__dict__` (`obj.__dict__['attr']`)
3. Non-data descriptors and class attributes from `type(obj).__mro__`
4. Raise `AttributeError`

**The key insight**: Non-data descriptors and regular class attributes are at the same priority level - both are found by walking the Method Resolution Order (MRO).

### Complete Example

```python
class DataDescriptor:
    def __get__(self, instance, owner):
        return "data descriptor"
    def __set__(self, instance, value):
        pass


class NonDataDescriptor:
    def __get__(self, instance, owner):
        return "non-data descriptor"


class Demo:
    data_desc = DataDescriptor()           # Priority 1
    non_data_desc = NonDataDescriptor()    # Priority 3
    class_attr = "class attribute"         # Priority 3


obj = Demo()

# Add instance attributes
obj.__dict__['data_desc'] = "instance dict"      # Priority 2
obj.__dict__['non_data_desc'] = "instance dict"  # Priority 2
obj.__dict__['class_attr'] = "instance dict"     # Priority 2

print(obj.data_desc)      # "data descriptor" (priority 1 wins)
print(obj.non_data_desc)  # "instance dict" (priority 2 wins over 3)
print(obj.class_attr)     # "instance dict" (priority 2 wins over 3)

# Remove instance attributes to see class level
del obj.__dict__['non_data_desc']
del obj.__dict__['class_attr']

print(obj.non_data_desc)  # "non-data descriptor" (priority 3)
print(obj.class_attr)     # "class attribute" (priority 3)
```

### Visual Summary

Priority 1: Data descriptors (has `__get__` + `__set__`/`__delete__`)
            Examples: `@property`, `ValidatedAttribute`
            ↓
Priority 2: Instance `__dict__`
            `obj.__dict__['attr']`
            ↓
Priority 3: Non-data descriptors (has only `__get__`)
            + Regular class attributes
            Examples: methods, classmethod, staticmethod, plain values
            ↓
Priority 4: `AttributeError`

### Practical Implication

This is why you can't shadow a property by doing `obj.x = value` (property is a data descriptor), but you can shadow a method or class variable (they're non-data descriptors/class attributes):

```python
class Example:
    class_var = 42

    @property
    def prop(self):
        return "property"

    def method(self):
        return "method"

obj = Example()

obj.class_var = 99    # Creates instance attribute, shadows class variable ✓
obj.method = "new"    # Creates instance attribute, shadows method ✓
obj.prop = "new"      # Calls property.__set__, raises AttributeError! ✗
```

This hierarchy is fundamental to how Python's object model works!

## Implementation of a descriptor class StaticMethod that emulates a static method

```python
class StaticMethod:
    def __init__(self, f):
        self._f = f

    def __get__(self, obj, objtype=None):
        return self._f
```

## What happens when setting attributes

Attribute setting (assignment) has a different, simpler lookup order than retrieval.

When setting an attribute (e.g., instance.attr = value), the order is:

1. Data descriptors from the type (class) and its bases
2. Instance `__dict__`

That's it. If a data descriptor with `__set__` is found in the class, it handles the assignment. Otherwise, the value goes directly into the instance dictionary.

### Key difference from retrieval

Non-data descriptors are not checked at all during attribute setting, because they don't define `__set__`. This is why you can easily override methods (non-data descriptors) by assigning to the instance:

```python
class Example:
    def method(self):
        return "method"

obj = Example()
obj.method = "overridden"  # Goes straight to obj.__dict__
print(obj.method)  # "overridden"
```

But data descriptors intercept the assignment:

```python
class Example:
    @property
    def x(self):
        return self._x

    @x.setter
    def x(self, value):
        self._x = value

obj = Example()
obj.x = 42  # Calls the descriptor's __set__ method
obj.__dict__['x'] = 99  # Won't affect obj.x
print(obj.x)  # 42 - descriptor still in control
```

So: **get** checks data descriptors → instance dict → non-data descriptors, but **set** only checks data descriptors → instance dict.

## What happens when deleting attributes

Excellent question! Attribute deletion (e.g., `del instance.attr`) follows a similar pattern to setting:

1. Descriptors with `__delete__` from the type (class) and its bases
2. Instance `__dict__`

If a descriptor defining `__delete__` is found in the class, it handles the deletion. Otherwise, Python tries to delete from the instance dictionary.

Important note: A descriptor with `__get__` and `__delete__` (but no `__set__`) is still considered a data descriptor for the purposes of attribute lookup priority.

Example:

```python
class DeleteControl:
    def __get__(self, instance, owner):
        return instance._value

    def __delete__(self, instance):
        print("Descriptor controlling deletion")
        del instance._value

class Example:
    x = DeleteControl()

obj = Example()
obj._value = 42
obj.__dict__['x'] = 99  # Instance dict has 'x'

print(obj.x)  # 99 - instance dict wins (DeleteControl is data descriptor for lookup)
del obj.x     # "Descriptor controlling deletion" - descriptor intercepts
```

Summary of all three operations:

- Get: data descriptors → instance dict → non-data descriptors → AttributeError
- Set: data descriptors (with `__set__`) → instance dict
- Delete: descriptors (with `__delete__`) → instance dict → AttributeError

## Concise but illustrative example which shows a use case for writing your own descriptor

A concise example that demonstrates a practical use case for descriptors - validating attribute values:

```python
class ValidatedAttribute:
    """A descriptor that validates values before setting them."""

    def __init__(self, min_value=None, max_value=None):
        self.min_value = min_value
        self.max_value = max_value

    def __set_name__(self, owner, name):
        self.name = f"_{name}"

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return getattr(instance, self.name, None)

    def __set__(self, instance, value):
        if self.min_value is not None and value < self.min_value:
            raise ValueError(f"Value must be >= {self.min_value}")
        if self.max_value is not None and value > self.max_value:
            raise ValueError(f"Value must be <= {self.max_value}")
        setattr(instance, self.name, value)


# Usage
class Temperature:
    celsius = ValidatedAttribute(min_value=-273.15)  # Absolute zero

    def __init__(self, celsius):
        self.celsius = celsius


# Demo
t = Temperature(25)
print(t.celsius)  # 25

t.celsius = -300  # Raises ValueError: Value must be >= -273.15
```

Why this works well for a talk:

1. **Clear problem**: You need validation logic that's reusable across multiple attributes/classes
2. **DRY principle**: Without descriptors, you'd need to write validation in every `__init__` or property setter
3. **Declarative syntax**: The validation rules are clear at the class level
4. **Reusability**: The same descriptor can validate age, temperature, price, etc.

## A caching example - 2-D point which stores `x` & `y`, but calculates the polar coordinates `r` and `theta`

### A caching descriptor example for the 2D point use case

```python
import math


class CachedProperty:
    """A descriptor that caches computed values and invalidates when dependencies change."""

    def __init__(self, compute_func, *dependencies):
        self.compute_func = compute_func
        self.dependencies = dependencies

    def __set_name__(self, owner, name):
        self.name = name
        self.cache_name = f"_cache_{name}"

    def __get__(self, instance, owner):
        if instance is None:
            return self

        # Check if cache is valid
        if not hasattr(instance, self.cache_name):
            # Compute and cache the value
            value = self.compute_func(instance)
            setattr(instance, self.cache_name, value)

        return getattr(instance, self.cache_name)

    def invalidate(self, instance):
        """Remove cached value."""
        if hasattr(instance, self.cache_name):
            delattr(instance, self.cache_name)


class InvalidatingAttribute:
    """An attribute that invalidates cached properties when set."""

    def __init__(self):
        self.cached_properties = []

    def __set_name__(self, owner, name):
        self.name = f"_{name}"
        # Find all CachedProperty descriptors that depend on this attribute
        for attr_name in dir(owner):
            attr = getattr(owner, attr_name, None)
            if isinstance(attr, CachedProperty) and name in attr.dependencies:
                self.cached_properties.append(attr)

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return getattr(instance, self.name)

    def __set__(self, instance, value):
        setattr(instance, self.name, value)
        # Invalidate all dependent cached properties
        for cached_prop in self.cached_properties:
            cached_prop.invalidate(instance)


class Point:
    x = InvalidatingAttribute()
    y = InvalidatingAttribute()

    # Cached polar coordinates
    r = CachedProperty(lambda self: math.sqrt(self.x**2 + self.y**2), 'x', 'y')
    theta = CachedProperty(lambda self: math.atan2(self.y, self.x), 'x', 'y')

    def __init__(self, x, y):
        self.x = x
        self.y = y


# Demo
p = Point(3, 4)
print(f"First access to r: {p.r}")  # Calculates: 5.0
print(f"Second access to r: {p.r}")  # Cached: 5.0
print(f"Theta: {p.theta}")  # Calculates: 0.927...

p.x = 5  # Invalidates cache
print(f"After changing x, r: {p.r}")  # Recalculates: 6.403...
```

Key points for talk:

1. **Lazy evaluation**: Polar coordinates aren't computed until needed
2. **Automatic caching**: Expensive `sqrt()` and `atan2()` calls only happen once
3. **Smart invalidation**: Cache automatically clears when `x` or `y` change
4. **Declarative**: Dependencies are explicit at the class level

### Simpler alternative

```python
class CachedProperty:
    """Simple cached property descriptor."""

    def __init__(self, compute_func):
        self.compute_func = compute_func

    def __set_name__(self, owner, name):
        self.name = f"_cache_{name}"

    def __get__(self, instance, owner):
        if instance is None:
            return self
        if not hasattr(instance, self.name):
            setattr(instance, self.name, self.compute_func(instance))
        return getattr(instance, self.name)


class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    @CachedProperty
    def r(self):
        print("Computing r...")
        return math.sqrt(self.x**2 + self.y**2)

    @CachedProperty
    def theta(self):
        print("Computing theta...")
        return math.atan2(self.y, self.x)
```

This simpler version shows the caching concept without the invalidation complexity.

## Does this still work if `x` and `y` are slots and not regular attributes

No, it doesn't work as-is with `__slots__` because the descriptors try to store data on the instance using `setattr(instance, self.name, value)`, but `__slots__` prevents arbitrary attribute
   assignment.

### The Problem

```python
class Point:
    __slots__ = ('x', 'y', '__dict__')  # Only these attributes allowed!

    x = InvalidatingAttribute()  # Tries to store as '_x'
    y = InvalidatingAttribute()  # Tries to store as '_y'

    r = CachedProperty(lambda self: math.sqrt(self.x**2 + self.y**2), 'x', 'y') # Tries to store cache as '_cache_r'
    theta = CachedProperty(lambda self: math.atan2(self.y, self.x), 'x', 'y') # Tries to store cache as '_cache_theta'

    def __init__(self, x, y):
        self.x = x  # Descriptor tries: setattr(instance, '_x', value)
        self.y = y  # Fails! '_x' and '_y' not in __slots__
```

### Solution 1: Include descriptor storage in `__slots__`

```python
class Point:
    __slots__ = ('_x', '_y', '_cache_r', '_cache_theta')  # All storage slots

    x = InvalidatingAttribute()
    y = InvalidatingAttribute()
    r = CachedProperty(lambda self: math.sqrt(self.x**2 + self.y**2), 'x', 'y')
    theta = CachedProperty(lambda self: math.atan2(self.y, self.x), 'x', 'y')

    def __init__(self, x, y):
        self.x = x
        self.y = y
```

This works, but you have to remember to add every internal storage name to `__slots__`.

### Solution 2: Store data in the descriptor itself (with WeakKeyDictionary)

```python
import weakref
import math


class InvalidatingAttribute:
    """Stores data in the descriptor, not on the instance."""

    def __init__(self):
        self.data = weakref.WeakKeyDictionary()
        self.cached_properties = []

    def __set_name__(self, owner, name):
        # Find dependent cached properties
        for attr_name in dir(owner):
            attr = getattr(owner, attr_name, None)
            if isinstance(attr, CachedProperty) and name in attr.dependencies:
                self.cached_properties.append(attr)

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return self.data.get(instance)

    def __set__(self, instance, value):
        self.data[instance] = value
        # Invalidate dependent caches
        for cached_prop in self.cached_properties:
            cached_prop.invalidate(instance)


class CachedProperty:
    """Stores cached values in the descriptor."""

    def __init__(self, compute_func, *dependencies):
        self.compute_func = compute_func
        self.dependencies = dependencies
        self.cache = weakref.WeakKeyDictionary()

    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, instance, owner):
        if instance is None:
            return self

        if instance not in self.cache:
            self.cache[instance] = self.compute_func(instance)

        return self.cache[instance]

    def invalidate(self, instance):
        self.cache.pop(instance, None)


class Point:
    __slots__ = ()  # No instance attributes needed!

    x = InvalidatingAttribute()
    y = InvalidatingAttribute()
    r = CachedProperty(lambda self: math.sqrt(self.x**2 + self.y**2), 'x', 'y')
    theta = CachedProperty(lambda self: math.atan2(self.y, self.x), 'x', 'y')

    def __init__(self, x, y):
        self.x = x
        self.y = y


# Demo
p = Point(3, 4)
print(f"r: {p.r}")  # 5.0
print(f"r again: {p.r}")  # Cached
p.x = 5
print(f"r after change: {p.r}")  # Recalculated: 6.403...
```

### Key Differences

  | Approach                          | Pros                                 | Cons                                             |
  |-----------------------------------|--------------------------------------|--------------------------------------------------|
  | Solution 1 (explicit `__slots__`) | Simple, fast                         | Must list all internal names                     |
  | Solution 2 (WeakKeyDictionary)    | Clean `__slots__`, automatic cleanup | Slightly slower, uses more memory per descriptor |

## Explanation of the internal workings of `__slots__` descriptors

### How `__slots__` Works Internally

When you define `__slots__`, Python:

1. Creates slot descriptors - one for each name in `__slots__`
2. Allocates a fixed-size array on each instance (instead of a `__dict__`)
3. Each slot descriptor knows its index in that array

### Demonstration

```python
class Point:
    __slots__ = ('x', 'y')

# What did Python create?
print(type(Point.x))  # <class 'member_descriptor'>
print(type(Point.y))  # <class 'member_descriptor'>

# These are DATA DESCRIPTORS
print(hasattr(Point.x, '__get__'))  # True
print(hasattr(Point.x, '__set__'))  # True
print(hasattr(Point.x, '__delete__'))  # True

# Create an instance
p = Point()
print(hasattr(p, '__dict__'))  # False - no dict!

# The slot descriptors manage storage
p.x = 3  # Calls Point.x.__set__(p, 3)
p.y = 4  # Calls Point.y.__set__(p, 4)

print(p.x)  # Calls Point.x.__get__(p, Point) -> 3
```

### How Slot Descriptors Work

```python
# Pseudocode of what Python does internally:

class member_descriptor:
    """Built-in descriptor type for __slots__"""

    def __init__(self, name, offset):
        self.name = name
        self.offset = offset  # Index in the instance's slot array

    def __get__(self, instance, owner):
        if instance is None:
            return self
        # Read from instance's slot array at this offset
        value = instance._slotvalues[self.offset]
        if value is UNSET:
            raise AttributeError(f"'{owner.__name__}' object has no attribute '{self.name}'")
        return value

    def __set__(self, instance, value):
        # Write to instance's slot array at this offset
        instance._slotvalues[self.offset] = value

    def __delete__(self, instance):
        # Mark as unset in the slot array
        instance._slotvalues[self.offset] = UNSET


# When you define:
class Point:
    __slots__ = ('x', 'y')

# Python roughly does:
# Point.x = member_descriptor('x', offset=0)
# Point.y = member_descriptor('y', offset=1)
# And each Point instance gets: _slotvalues = [UNSET, UNSET]
```

### Visual Representation

```python
class Point:
    __slots__ = ('x', 'y')

p1 = Point()
p2 = Point()

# Class level - shared descriptors
# Point.x -> member_descriptor(name='x', offset=0)
# Point.y -> member_descriptor(name='y', offset=1)

# Instance level - each has its own array
# p1._slotvalues = [UNSET, UNSET]  (conceptually)
# p2._slotvalues = [UNSET, UNSET]

p1.x = 3
p1.y = 4
# p1._slotvalues = [3, 4]

p2.x = 10
p2.y = 20
# p2._slotvalues = [10, 20]
```

### Why This Is Important

1. **Memory efficiency**: Fixed array vs. dict (no hash table overhead)
2. **Faster attribute access**: Direct array index vs. hash lookup
3. **They're data descriptors**: Slot descriptors always win over instance `__dict__` (if you include `'__dict__'` in `__slots__`)

### What Happens When You Override

```python
class Point:
    __slots__ = ('x', 'y')

    x = MyDescriptor()  # This REPLACES the slot descriptor!

# Now:
# Point.x = MyDescriptor() (your custom descriptor)
# Point.y = member_descriptor(...) (still the slot descriptor)

p = Point()
# The instance still has the slot array: [UNSET, UNSET]
# But slot 0 is now orphaned - nothing uses it!
# When you access p.x, it calls MyDescriptor.__get__, not the slot

p.y = 4  # Uses the slot descriptor (offset=1)
p.x = 3  # Uses MyDescriptor, NOT the slot!
```

### Proving Slot Descriptors Exist

```python
class Point:
    __slots__ = ('x', 'y')

# Access the slot descriptor directly
x_descriptor = Point.__dict__['x']
print(x_descriptor)  # <member 'x' of 'Point' objects>

p = Point()

# Use the descriptor manually
x_descriptor.__set__(p, 100)
print(x_descriptor.__get__(p, Point))  # 100
print(p.x)  # 100 (same thing)

# Delete using the descriptor
x_descriptor.__delete__(p)
try:
    print(p.x)
except AttributeError as e:
    print(e)  # 'Point' object has no attribute 'x'
```

### Key Takeaway

`__slots__` creates data descriptors on the class that manage a fixed-size array on each instance. When you define a class attribute with the same name as a slot, you replace that slot descriptor with your own attribute/descriptor, making that slot storage inaccessible!

This is fundamental to understanding why your InvalidatingAttribute and CachedProperty don't work well with `__slots__` without including either:

- The internal names (`'_x'`, `'_y'`, `'_cache_r'`, etc.) in `__slots__`, OR
- `'__dict__'` in `__slots__`, OR
- Using WeakKeyDictionary in the descriptor itself

## Beyond the descriptor protocol - other special methods involved in attribute lookup and resolution

There are several special methods that interact with attribute lookup. Here's the complete picture:

### Special Methods for Attribute Access

#### 1. `__getattribute__` (Always Called First)

Called for EVERY attribute access - even before the descriptor protocol runs.

```python
class LoggedAccess:
    def __init__(self):
        self.value = 42

    def __getattribute__(self, name):
        print(f"Accessing: {name}")
        # Must call super() to get normal behavior
        return super().__getattribute__(name)

obj = LoggedAccess()
print(obj.value)
# Output:
# Accessing: value
# 42
```

#### 2. `__getattr__` (Fallback)

Called ONLY when normal lookup fails (after descriptors, instance `__dict__`, class attributes).

```python
class Fallback:
    def __getattr__(self, name):
        return f"No attribute '{name}' found, returning default"

obj = Fallback()
obj.real_attr = 100

print(obj.real_attr)  # 100 (normal lookup)
print(obj.missing)    # "No attribute 'missing' found, returning default"
```

#### 3. `__setattr__` (Assignment)

Called for ALL attribute assignments.

```python
class ValidatedSet:
    def __setattr__(self, name, value):
        print(f"Setting {name} = {value}")
        super().__setattr__(name, value)

obj = ValidatedSet()
obj.x = 42
# Output: Setting x = 42
```

#### 4. `__delattr__` (Deletion)

Called for ALL attribute deletions.

```python
class ProtectedDelete:
    def __init__(self):
        self.x = 42

    def __delattr__(self, name):
        if name == 'x':
            raise AttributeError(f"Cannot delete {name}")
        super().__delattr__(name)

obj = ProtectedDelete()
del obj.x  # Raises: AttributeError: Cannot delete x
```

### Complete Attribute Lookup Flow

#### For `obj.attr` (getting)

1. `obj.__getattribute__('attr')` is called
    ↓ (inside `__getattribute__`)
2. Look for data descriptor in `type(obj).__mro__`
    ↓ (if found, call `descriptor.__get__`)
3. Look in `obj.__dict__`
    ↓
4. Look for non-data descriptor or class attribute in `type(obj).__mro__`
    ↓ (if found and is descriptor, call `descriptor.__get__`)
5. If nothing found, call `obj.__getattr__('attr')`
    ↓
6. If `__getattr__` not defined or also fails, raise `AttributeError`

#### For `obj.attr = value` (setting)

1. `obj.__setattr__('attr', value)` is called
    ↓ (inside `__setattr__`)
2. Look for data descriptor in `type(obj).__mro__`
    ↓ (if found and has `__set__`, call `descriptor.__set__`)
3. Otherwise, set `obj.__dict__['attr'] = value`

#### For `del obj.attr` (deleting)

1. `obj.__delattr__('attr')` is called
    ↓ (inside `__delattr__`)
2. Look for data descriptor in `type(obj).__mro__`
    ↓ (if found and has `__delete__`, call `descriptor.__delete__`)
3. Otherwise, `del obj.__dict__['attr']`

### How They All Interact

```python
class MyDescriptor:
    def __get__(self, instance, owner):
        print("  MyDescriptor.__get__")
        return "descriptor value"

    def __set__(self, instance, value):
        print("  MyDescriptor.__set__")


class Demo:
    desc = MyDescriptor()

    def __getattribute__(self, name):
        print(f"__getattribute__({name})")
        return super().__getattribute__(name)

    def __getattr__(self, name):
        print(f"__getattr__({name})")
        return "fallback"

    def __setattr__(self, name, value):
        print(f"__setattr__({name}, {value})")
        super().__setattr__(name, value)


obj = Demo()

print("\n--- Getting descriptor ---")
result = obj.desc
# Output:
# __getattribute__(desc)
#   MyDescriptor.__get__

print("\n--- Setting descriptor ---")
obj.desc = 100
# Output:
# __setattr__(desc, 100)
#   MyDescriptor.__set__

print("\n--- Getting missing attribute ---")
result = obj.missing
# Output:
# __getattribute__(missing)
# __getattr__(missing)

print("\n--- Setting regular attribute ---")
obj.regular = 42
# Output:
# __setattr__(regular, 42)
```

### Practical Example: Implementing a Proxy

```python
class Proxy:
    """Forwards all attribute access to another object."""

    def __init__(self, target):
        # Use object.__setattr__ to avoid infinite recursion
        object.__setattr__(self, '_target', target)

    def __getattribute__(self, name):
        # Avoid infinite recursion for _target
        if name == '_target':
            return object.__getattribute__(self, '_target')

        target = object.__getattribute__(self, '_target')
        return getattr(target, name)

    def __setattr__(self, name, value):
        target = object.__getattribute__(self, '_target')
        setattr(target, name, value)


class Original:
    def __init__(self):
        self.x = 42

    def method(self):
        return "called method"


original = Original()
proxy = Proxy(original)

print(proxy.x)  # 42
print(proxy.method())  # "called method"
proxy.x = 100
print(original.x)  # 100 (modified the original!)
```

### Common Pitfall: Infinite Recursion

```python
class Bad:
    def __getattribute__(self, name):
        print(f"Getting {name}")
        return self.name  # Recursion! Calls __getattribute__ again!

obj = Bad()
# obj.x  # RecursionError!


class Good:
    def __getattribute__(self, name):
        print(f"Getting {name}")
        # Use super() or object.__getattribute__
        return super().__getattribute__(name)
```

### Summary

| Method                  | When Called                            | Use Case                                    |
|-------------------------|----------------------------------------|---------------------------------------------|
| `__getattribute__`      | Every attribute access                 | Logging, proxies, computed attributes       |
| `__getattr__`           | When normal lookup fails               | Fallback values, lazy loading, delegation   |
| `__setattr__`           | Every attribute assignment             | Validation, type checking, observers        |
| `__delattr__`           | Every attribute deletion               | Protection, cleanup hooks                   |
| Descriptor `__get__`    | Via `__getattribute__`                 | Lazy computation, validation, type checking |
| Descriptor `__set__`    | Via `__setattr__`                      | Validation, type checking, observers        |
| Descriptor `__delete__` | Via `__delattr__`                      | Cleanup, protection                         |

**Key insight**: `__getattribute__` and `__setattr__` are the "outer layer" that orchestrate the entire lookup process, including calling into the descriptor protocol. Descriptors are more surgical (per-attribute), while `__getattribute__`/`__setattr__` affect ALL attributes.

## Other Questions

Here are some important topics that would strengthen the conference talk:

### 1. `__set_name__` (Python 3.6+)

We used it but didn't explain it. This method is called automatically when the class is created:

```python
class MyDescriptor:
    def __set_name__(self, owner, name):
        # owner = the class being defined
        # name = the attribute name this descriptor is assigned to
        print(f"Bound to {owner.__name__}.{name}")
        self.name = name


class Demo:
    x = MyDescriptor()  # Calls __set_name__(Demo, 'x')
    y = MyDescriptor()  # Calls __set_name__(Demo, 'y')

# Output:
# Bound to Demo.x
# Bound to Demo.y
```

Before `__set_name__` (Python < 3.6), you had to manually pass the name:

```python
# Old way
class Demo:
    x = MyDescriptor('x')  # Had to repeat the name!
    y = MyDescriptor('y')
```

### 2. Functions are Descriptors! (Method Binding)

This is THE KILLER EXAMPLE for the talk - people use this every day without knowing:

```python
class Demo:
    def method(self):
        return "called"

obj = Demo()

# What is Demo.method?
print(Demo.method)  # <function Demo.method at 0x...>
print(type(Demo.method))  # <class 'function'>

# What is obj.method?
print(obj.method)  # <bound method Demo.method of <Demo object>>
print(type(obj.method))  # <class 'method'>

# Functions implement __get__!
print(hasattr(Demo.method, '__get__'))  # True
```

#### How it works

```python
import types

# When you define a function in a class, it's a descriptor!
class Demo:
    def method(self):
        return f"self is {self}"

obj = Demo()

# These are equivalent:
obj.method()
Demo.method(obj)

# What's really happening:
Demo.__dict__['method'].__get__(obj, Demo)()

# The function's __get__ returns a bound method
bound = Demo.method.__get__(obj, Demo)
print(bound)  # <bound method Demo.method of <Demo object>>
print(bound())  # "self is <Demo object>"
```

#### Implementing method binding yourself

```python
class Function:
    """Simplified function descriptor showing method binding."""

    def __init__(self, func):
        self.func = func

    def __get__(self, instance, owner):
        if instance is None:
            return self  # Unbound: Demo.method
        # Return a bound method
        return lambda *args, **kwargs: self.func(instance, *args, **kwargs)


class Demo:
    @Function
    def method(self, x):
        return f"self={self}, x={x}"

obj = Demo()
print(obj.method(42))  # self=<Demo object>, x=42
```

### 3. classmethod and staticmethod as Descriptors

```python
class classmethod_descriptor:
    """How classmethod works internally."""

    def __init__(self, func):
        self.func = func

    def __get__(self, instance, owner):
        # Always bind to the class, not the instance
        return lambda *args, **kwargs: self.func(owner, *args, **kwargs)


class staticmethod_descriptor:
    """How staticmethod works internally."""

    def __init__(self, func):
        self.func = func

    def __get__(self, instance, owner):
        # Return the function unchanged (no binding)
        return self.func


class Demo:
    @classmethod_descriptor
    def class_method(cls, x):
        return f"cls={cls}, x={x}"

    @staticmethod_descriptor
    def static_method(x):
        return f"x={x}"

obj = Demo()
print(obj.class_method(42))  # cls=<class 'Demo'>, x=42
print(obj.static_method(42))  # x=42
```

### 4. The owner Parameter in `__get__`

```python
class Demo:
    def __get__(self, instance, owner):
        # instance: the instance (obj), or None if accessed from class
        # owner: the class (always provided)
        if instance is None:
            return f"Accessed from {owner.__name__} class"
        return f"Accessed from instance of {owner.__name__}"


class MyClass:
    desc = Demo()

print(MyClass.desc)  # "Accessed from MyClass class"
print(MyClass().desc)  # "Accessed from instance of MyClass"
```

Use `owner` when you need the class regardless of whether accessed from class or instance:

```python
class TypedAttribute:
    def __init__(self, expected_type):
        self.expected_type = expected_type

    def __set_name__(self, owner, name):
        self.name = name

    def __get__(self, instance, owner):
        if instance is None:
            # Return helpful info when accessed from class
            return f"<{self.expected_type.__name__} attribute>"
        return getattr(instance, f'_{self.name}')

    def __set__(self, instance, value):
        if not isinstance(value, self.expected_type):
            raise TypeError(f"Expected {self.expected_type.__name__}")
        setattr(instance, f'_{self.name}', value)


class Person:
    age = TypedAttribute(int)

print(Person.age)  # "<int attribute>" (helpful!)
```

### 5. When to Use What

| Use Case                        | Use This           | Why                |
|---------------------------------|--------------------|--------------------|
| Single computed attribute       | `@property`        | Simple, readable   |
| Reusable attribute logic        | Descriptor         | DRY across classes |
| Type validation for many attrs  | Descriptor         | Declarative        |
| Log/intercept ALL attributes    | `__getattribute__` | Affects everything |
| Fallback for missing attributes | `__getattr__`      | Dynamic attrs      |
| ORM column definitions          | Descriptor         | SQLAlchemy, Django |

```python
# Just one computed attribute? Use @property
class Point:
    @property
    def r(self):
        return math.sqrt(self.x**2 + self.y**2)

# Reusable across many attributes/classes? Use descriptor
class Validated:
    age = IntRange(0, 150)
    score = IntRange(0, 100)
    rating = IntRange(1, 5)
```

### 6. Descriptors on Instances Don't Work

```python
class MyDescriptor:
    def __get__(self, instance, owner):
        return "descriptor"

# On the class - works!
class Demo:
    x = MyDescriptor()

print(Demo().x)  # "descriptor" ✓

# On the instance - doesn't work!
obj = Demo()
obj.y = MyDescriptor()
print(obj.y)  # <MyDescriptor object> ✗ (not "descriptor")
```

Why? Descriptors must be on the class, not the instance. The lookup process checks `type(obj).__dict__`, not `obj.__dict__`.

### 7. Real-World Examples

```python
# Django ORM
class User(models.Model):
    name = models.CharField(max_length=100)  # Descriptor!
    age = models.IntegerField()  # Descriptor!

# SQLAlchemy
class User(Base):
    name = Column(String)  # Descriptor!
    age = Column(Integer)  # Descriptor!

# Pydantic (uses descriptors internally)
class User(BaseModel):
    name: str
    age: int
```

### 8. Property is Just a Descriptor

```python
# These are equivalent:

# Using @property
class Point:
    @property
    def r(self):
        return math.sqrt(self.x**2 + self.y**2)

# Using a descriptor
class computed:
    def __init__(self, func):
        self.func = func

    def __get__(self, instance, owner):
        if instance is None:
            return self
        return self.func(instance)

class Point:
    @computed
    def r(self):
        return math.sqrt(self.x**2 + self.y**2)
```
