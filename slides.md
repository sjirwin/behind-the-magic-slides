<!-- .slide: class="center" -->
# Behind the Magic
### Unlocking Python's Descriptor Protocol

<center>
<br/>Scott Irwin
<p>&nbsp;
<img src="images/BBGEngineering_black.png"
     style="border: none; box-shadow: none; height: 100px"
     alt="Bloomberg Engineering logo"
/>
</center>

------

## About Me

- Bloomberg Engineering
  - Joined in 2014 as Senior Engineer
  - Roles: Individual Contributor, Team Lead, and Technical Trainer
  - Python Guild Leader since 2018
    - Former Co-chair
- Python educator

------

## Why This Talk

- Curiosity!
- Descriptors are everywhere

===

<!-- .slide: class="center" -->
# Descriptors

------

## What is a descriptor

- A **descriptor** is any object that implements the descriptor protocol
- The **descriptor protocol** controls attribute access

------

## Descriptor Protocol

- Enables objects that are attributes of other classes to customize their behavior when they are read or modified
- Protocol methods
  - `__get__`
  - `__set__`
  - `__delete__`

------

## Descriptor Types

<div style="display: flex; gap: 20px;">
<div style="margin-left: 10%;">

#### Non-Data Descriptors

- Defines only `__get__`

</div>

---

<div style="margin-right: 10%;">

#### Data Descriptors

- Defines one or both of:
  - `__set__`
  - `__delete__`
- Also typically defines `__get__`

</div>
</div>

------

## Descriptors Are Everywhere

<div style="display: flex; gap: 20px;">
<div style="margin-left: 10%;">

#### Non-Data Descriptors

-  Class methods
- `@classmethod`
- `@staticmethod`
- Methods on built-in types
  - E.g., `list.append`, `dict.items`
- `super()`

</div>

---

<div style="margin-right: 10%;">

#### Data Descriptors

- `@property`
- `__slots__` attributes

</div>
</div>

===

<!-- .slide: class="center" -->
# Attribute Lookup
## Basic View

------

## Lookup Order - Simple View

For `obj.attr`

1. Data descriptors from `type(obj)` (class)
1. `obj.__dict__['attr']` (instance `__dict__`)
1. Non-data descriptors from `type(obj)`
1. Raise `AttributeError`

------

## Lookup Order - Expanded View

For `obj.attr`

1. Data descriptors from `type(obj).__mro__` (class and its bases)
1. `obj.__dict__['attr']` (instance `__dict__`)
1. Non-data descriptors and class attributes from `type(obj).__mro__`
1. Raise `AttributeError`

===

<!-- .slide: class="center" -->
# Live Demo

===

<!-- .slide: class="center" -->
# Attribute Lookup
## Full View

------

## Lookup Order - Get

For `obj.attr`

1. `obj.__getattribute__('attr')`
<br/>&nbsp;&#8595; (inside `__getattribute__`)
1. Data descriptor from `type(obj).__mro__`
1. `obj.__dict__['attr']`
1. Non-data descriptors and class attributes from `type(obj).__mro__`
1. Call `obj.__getattr__('attr')`
1. Raise `AttributeError` (`__getattr__` not defined or also fails)

------

## Lookup Order - Set

For `obj.attr = value`

1. `obj.__setattr__('attr', value)`
<br/>&nbsp;&#8595; (inside `__setattr__`)
1. Data descriptor from `type(obj).__mro__`
1. `obj.__dict__['attr'] = value`
1. Raise `AttributeError` (no `__dict__`)

------

## Lookup Order - Del

For `del obj.attr`

1. `obj.__delattr__('attr')`
<br/>&nbsp;&#8595; (inside `__delattr__`)
1. Data descriptor from `type(obj).__mro__`
1. `del obj.__dict__['attr']`
1. Raise `AttributeError`

------

## Key Points

- `__getattribute__`, `__setattr__`, `__delattr__`
  - The "outer layer"
  - Affect **all** attributes
  - Orchestrate the lookup process, including calling the descriptor protocol
- Descriptors are per-attribute so more targeted

===

<!-- .slide: class="center" -->
# Writing Your Own Descriptors

------

## Some Uses For Descriptors

- Data Validation
- Lazy Evaluation & Caching
- Type Validation
- Logging / Debugging Attribute Access
- Unit Conversion
- Attribute Aliasing
- ORM Field Definitions

===

<!-- .slide: class="center" -->
# Wrapping Up

------

## How It All Connects

<div style="font-size: 0.80em;">

| Method                  | When Called                | Use Case                                    |
|-------------------------|----------------------------|---------------------------------------------|
| `__getattribute__`      | Every attribute access     | Logging, proxies, computed attributes       |
| `__getattr__`           | When normal lookup fails   | Fallback values, lazy loading, delegation   |
| `__setattr__`           | Every attribute assignment | Validation, type checking, observers        |
| `__delattr__`           | Every attribute deletion   | Protection, cleanup hooks                   |
| Descriptor `__get__`    | Via `__getattribute__`     | Lazy computation, validation, type checking |
| Descriptor `__set__`    | Via `__setattr__`          | Validation, type checking, observers        |
| Descriptor `__delete__` | Via `__delattr__`          | Cleanup, protection                         |

</div>

------

## Summary

- Descriptors are:
  - Everywhere
  - Essential to how Python works

------

<!-- .slide: class="center" -->
## Questions?

===

<!-- .slide: class="center" -->
# Appendices

===

<!-- .slide: class="center" -->
# Basic Demo

------

#### Non-Data Descriptor

<div style="display: flex; gap: 20px;">

```python
# descriptor.py

class SqueakyDescriptor:

    def __get__(self, obj, objtype=None):
        print(f"Squeak! Get value from {obj=} of type {objtype=}")
        if obj:
            return getattr(obj, "_squeaky", "Default")
        return getattr(objtype, "_squeaky", "Default")
```

---

```python
# demo.py

from descriptor import SqueakyDescriptor


class Demo:
    a = SqueakyDescriptor()
```

</div>

```python
$ python3.14 -i demo.py
>>> z = Demo()
>>> z.__dict__
{}
>>> z.a
Squeak! Get value from obj=<__main__.Demo object at 0x101c93380> of type objtype=<class '__main__.Demo'>
'Default'
>>> z.a = "hello"
>>> z.__dict__
{'a': 'hello'}
>>> z.a
'hello'
```

------

#### Data Descriptor (1)

<div style="display: flex; gap: 20px; font-size: 0.85em;">

```python [1,11-13]
# descriptor.py

class SqueakyDescriptor:

    def __get__(self, obj, objtype=None):
        print(f"Squeak! Get value from {obj=} of type {objtype=}")
        if obj:
            return getattr(obj, "_squeaky", "Default")
        return getattr(objtype, "_squeaky", "Default")
  
    def __set__(self, obj, value):
        print(f"Squeak! Set {value=} on {obj=}")
        setattr(obj, "_squeaky", value)

    # def __delete__(self, obj): ...
```

---

```python [1]
# demo.py

from descriptor import SqueakyDescriptor


class Demo:
    a = SqueakyDescriptor()
```

</div>

<div style="font-size: 1.00em;">

```python
$ python3.14 -i demo.py
>>> z = Demo()
>>> z.__dict__
{}
>>> z.a
Squeak! Get value from obj=<__main__.Demo object at 0x101883380> of type objtype=<class '__main__.Demo'>
'Default'
>>> z.a = "hello"
Squeak! Set value='hello' on obj=<__main__.Demo object at 0x101883380>
>>> z.__dict__
{'_squeaky': 'hello'}
>>> z.a
Squeak! Get value from obj=<__main__.Demo object at 0x101883380> of type objtype=<class '__main__.Demo'>
'hello'
```

</div>

------

#### Data Descriptor (2)

<div style="display: flex; gap: 20px; font-size: 0.75em;">

```python [1]
# descriptor.py

class SqueakyDescriptor:

    def __get__(self, obj, objtype=None):
        print(f"Squeak! Get value from {obj=} of type {objtype=}")
        if obj:
            return getattr(obj, "_squeaky", "Default")
        return getattr(objtype, "_squeaky", "Default")
  
    def __set__(self, obj, value):
        print(f"Squeak! Set {value=} on {obj=}")
        setattr(obj, "_squeaky", value)

    # def __delete__(self, obj): ...
```


---

```python [1,8]
# demo.py

from descriptor import SqueakyDescriptor


class Demo:
    a = SqueakyDescriptor()
    b = SqueakyDescriptor()
```

</div>

<div style="font-size: 0.95em;">

```python
$ python3.14 -i demo.py
>>> z = Demo()
>>> z.a
Squeak! Get value from obj=<__main__.Demo object at 0x103c7b380> of type objtype=<class '__main__.Demo'>
'Default'
>>> z.a = "hello"
Squeak! Set value='hello' on obj=<__main__.Demo object at 0x103c7b380>
>>> z.__dict__
{'_squeaky': 'hello'}
>>> z.a
Squeak! Get value from obj=<__main__.Demo object at 0x103c7b380> of type objtype=<class '__main__.Demo'>
'hello'
>>> z.b = 42
Squeak! Set value=42 on obj=<__main__.Demo object at 0x103c7b380>
>>> z.a
Squeak! Get value from obj=<__main__.Demo object at 0x103c7b380> of type objtype=<class '__main__.Demo'>
42
>>> z.__dict__
{'_squeaky': 42}
```

</div>

------

#### Data Descriptor (3)

<div style="display: flex; gap: 20px; font-size: 0.65em;">

```python [1,5-7,12-13,17]
# descriptor.py

class SqueakyDescriptor:

    def __set_name__(self, owner, name):
        print(f"Squeak! Set name {name=} on {owner=}")
        self.name = f"_{name}"

    def __get__(self, obj, objtype=None):
        print(f"Squeak! Get value from {obj=} of type {objtype=}")
        if obj:
            return getattr(obj, self.name, "Default")
        return getattr(objtype, self.name, "Default")

    def __set__(self, obj, value):
        print(f"Squeak! Set {value=} on {obj=}")
        setattr(obj, self.name, value)

    # def __delete__(self, obj): ...
```

---

```python [1]
# demo.py

from descriptor import SqueakyDescriptor


class Demo:
    a = SqueakyDescriptor()
    b = SqueakyDescriptor()
```

</div>

<div style="font-size: 0.90em;">

```python
$ python3.14 -i demo.py
Squeak! Set name name='a' on owner=<class '__main__.Demo'>
Squeak! Set name name='b' on owner=<class '__main__.Demo'>
>>> z = Demo()
>>> z.a
Squeak! Get value from obj=<__main__.Demo object at 0x105a9b380> of type objtype=<class '__main__.Demo'>
'Default'
>>> z.a = "hello"
Squeak! Set value='hello' on obj=<__main__.Demo object at 0x105a9b380>
>>> z.b
Squeak! Get value from obj=<__main__.Demo object at 0x105a9b380> of type objtype=<class '__main__.Demo'>
'Default'
>>> z.b = 42
Squeak! Set value=42 on obj=<__main__.Demo object at 0x105a9b380>
>>> z.a
Squeak! Get value from obj=<__main__.Demo object at 0x105a9b380> of type objtype=<class '__main__.Demo'>
'hello'
>>> z.__dict__
{'_a': 'hello', '_b': 42}
```

</div>

===

<!-- .slide: class="center" -->
# Point

------

#### Point - polar as functions

<div style="display: flex; gap: 20px;">

```python []
# point.py

import math

class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def r(self):
        print("Computing r ...")
        return math.sqrt(self.x**2 + self.y**2)

    def theta(self):
        print("Computing theta ...")
        return math.atan2(self.y, self.x)
```

---

```python
$ python3.14 -i point.py
>>> p = Point(3, 4)
>>> p.r()
Computing r ...
5.0
>>> p.r()
Computing r ...
5.0
>>> p.x = 5
>>> p.r()
Computing r ...
6.4031242374328485
```

</div>

------

#### Point - polar as properties

<div style="display: flex; gap: 20px;">

```python [10-18]
# point.py

import math

class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    @property
    def r(self):
        print("Computing r ...")
        return math.sqrt(self.x**2 + self.y**2)

    @property
    def theta(self):
        print("Computing theta ...")
        return math.atan2(self.y, self.x)
```

---

```python
$ python3.14 -i point.py
>>> p = Point(3, 4)
>>> p.r
Computing r ...
5.0
>>> p.r
Computing r ...
5.0
>>> p.x = 5
>>> p.r
Computing r ...
6.4031242374328485
```

</div>

------

#### Point - polar as cached properties

<div style="display: flex; gap: 20px;">

```python [4, 11-19]
# point.py

import math
from functools import cached_property

class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    @cached_property
    def r(self):
        print("Computing r ...")
        return math.sqrt(self.x**2 + self.y**2)

    @cached_property
    def theta(self):
        print("Computing theta ...")
        return math.atan2(self.y, self.x)
```

---

```python
$ python3.14 -i point.py
>>> p = Point(3, 4)
>>> p.r
Computing r ...
5.0
>>> p.r
5.0
>>> p.x = 5
>>> p.r
5.0
```

</div>

------

#### Point - polar as cached properties (DIY)

<div style="display: flex; gap: 20px;">

```python [4,11,16]
# point.py

import math
from descriptor import CachedProperty

class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    @CachedProperty
    def r(self):
        print("Computing r ...")
        return math.sqrt(self.x**2 + self.y**2)

    @CachedProperty
    def theta(self):
        print("Computing theta ...")
        return math.atan2(self.y, self.x)
```

---

```python []
# descriptor.py

class CachedProperty:

    def __init__(self, compute_func):
        self.compute_func = compute_func

    def __set_name__(self, owner, name):
        self.name = f"_cache_{name}"

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        if not hasattr(obj, self.name):
            setattr(obj, self.name, self.compute_func(obj))
        return getattr(obj, self.name)
```

</div>

```python
$ python3.14 -i point.py
>>> p = Point(3, 4)
>>> p.r
Computing r ...
5.0
>>> p.r
5.0
>>> p.x = 5
>>> p.r
5.0
```

------

#### Point - polar as invalidating cached properties (1)

<div style="display: flex; gap: 20px; font-size=0.50em">

```python [5, 7, 10-11, 17-29]
# descriptor.py

class CachedProperty:

    def __init__(self, compute_func, *dependencies):
        self.compute_func = compute_func
        self.dependencies = dependencies

    def __set_name__(self, owner, name):
        self.name = name
        self.cache_name = f"_cache_{name}"

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self

        # Check if cache is valid
        if not hasattr(obj, self.cache_name):
            # Compute and cache the value
            print(f"Computing {self.name} ...")
            value = self.compute_func(obj)
            setattr(obj, self.cache_name, value)

        return getattr(obj, self.cache_name)

    def invalidate(self, obj):
        """Remove cached value."""
        if hasattr(obj, self.cache_name):
            delattr(obj, self.cache_name)
```

---

```python []
# descriptor.py

class InvalidatingAttribute:

    def __init__(self):
        self.cached_properties = []

    def __set_name__(self, owner, name):
        self.name = f"_{name}"
        # Find all CachedProperty descriptors
        # that depend on this attribute
        for attr_name in dir(owner):
            attr = getattr(owner, attr_name, None)
            if (isinstance(attr, CachedProperty) 
                and name in attr.dependencies):
                self.cached_properties.append(attr)

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.name)

    def __set__(self, obj, value):
        setattr(obj, self.name, value)
        # Invalidate all dependent cached properties
        for cached_prop in self.cached_properties:
            cached_prop.invalidate(obj)
```

</div>

------

#### Point - polar as invalidating cached properties (2)

<div style="display: flex; gap: 20px; font-size=0.50em">

```python [4, 7-12]
# point.py

import math
from descriptor import CachedProperty, InvalidatingAttribute

class Point:
    x = InvalidatingAttribute()
    y = InvalidatingAttribute()

    # Cached polar coordinates
    r = CachedProperty(lambda self: math.sqrt(self.x**2 + self.y**2), "x", "y")
    theta = CachedProperty(lambda self: math.atan2(self.y, self.x), "x", "y")

    def __init__(self, x, y):
        self.x = x
        self.y = y
```

---

```python
$ python3.14 -i point.py
>>> p = Point(3, 4)
>>> p.r
Computing r ...
5.0
>>> p.r
5.0
>>> p.x = 5
>>> p.r
Computing r ...
6.4031242374328485
```

</div>

===

<!-- .slide: class="center" -->
# Use Case

## Validation

------

#### Unvalidated Attributes

<div style="display: flex; gap: 20px;">

```python
class Region:
    def __init__(self, height, width, depth):
        self.height = height
        self.width = width
        self.depth = depth

    def __repr__(self):
        return f"Region({self.height}, {self.width}, {self.depth})"
```

---

```python
from region import Region

region = Region(1, 2, 3)
print(region)  # Region(1, 2, 3)

region.height = -42
print(region)  # Region(-42, 2, 3)
```

</div>

------

#### Validated Attributes - `@property`

<div style="display: flex; gap: 20px;">

```python
class Region:
    def __init__(self, height, width, depth):
        self.height = height
        self.width = width
        self.depth = depth
    def __repr__(self):
        return f"Region({self.height}, {self.width}, {self.depth})"
    @property
    def height(self):
        return self._height
    @height.setter
    def height(self, value):
        if value < 0:
            raise ValueError("Value must be >= 0")
        self._height = value
    @property
    def width(self):
        return self._width
    @width.setter
    def width(self, value):
        if value < 0:
            raise ValueError("Value must be >= 0")
        self._width = value
    @property
    def depth(self):
        return self._depth
    @depth.setter
    def depth(self, value):
        if value < 0:
            raise ValueError("Value must be >= 0")
        self._depth = value
```

---

```python
from region import Region

region = Region(1, 2, 3)
print(region)  # Region(1, 2, 3)

region.height = -42  # ValueError: Value must be >= 0
```

</div>

------

#### Validated via `@property` - Multiple Classes

<div style="display: flex; gap: 20px; font-size: 0.94em;">

```python
# region.py

class Region:
    def __init__(self, height, width, depth):
        self.height = height
        self.width = width
        self.depth = depth
    def __repr__(self):
        return f"Region({self.height}, {self.width}, {self.depth})"
    @property
    def height(self):
        return self._height
    @height.setter
    def height(self, value):
        if value < 0:
            raise ValueError("Value must be >= 0")
        self._height = value
    @property
    def width(self):
        return self._width
    @width.setter
    def width(self, value):
        if value < 0:
            raise ValueError("Value must be >= 0")
        self._width = value
    @property
    def depth(self):
        return self._depth
    @depth.setter
    def depth(self, value):
        if value < 0:
            raise ValueError("Value must be >= 0")
        self._depth = value
```

---

```python
# temperature.py

_ABSOLUTE_ZERO = -273.15

class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius

    @property
    def celsius(self):
        return self._celsius

    @celsius.setter
    def celsius(self, value):
        if value < _ABSOLUTE_ZERO:
            raise ValueError(f"Value must be >= {_ABSOLUTE_ZERO}")
        self._celsius = value
```

</div>

------

#### Descriptor - `ValidatedAttribute`

```python
# attribute.py

class ValidatedAttribute:
    """A descriptor that validates values before setting them."""

    def __init__(self, min_value=None, max_value=None):
        self.min_value = min_value
        self.max_value = max_value

    def __set_name__(self, owner, name):
        self.name = f"_{name}"

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        return getattr(obj, self.name, None)

    def __set__(self, obj, value):
        if self.min_value is not None and value < self.min_value:
            raise ValueError(f"Value must be >= {self.min_value}")
        if self.max_value is not None and value > self.max_value:
            raise ValueError(f"Value must be <= {self.max_value}")
        setattr(obj, self.name, value)
```

------

#### Using `ValidatedAttribute`

<div style="display: flex; gap: 20px;">

```python
# region.py

from attribute import ValidatedAttribute

class Region:
    height = ValidatedAttribute(min_value=0)
    width = ValidatedAttribute(min_value=0)
    depth = ValidatedAttribute(min_value=0)

    def __init__(self, height, width, depth):
        self.height = height
        self.width = width
        self.depth = depth

    def __repr__(self):
        return f"Region({self.height}, {self.width}, {self.depth})"
```

---

```python
# temperature.py

from attribute import ValidatedAttribute

class Temperature:
    celsius = ValidatedAttribute(min_value=-273.15)  # Absolute zero

    def __init__(self, celsius):
        self.celsius = celsius
```

</div>

<div style="display: flex; gap: 20px;">

```python
from region import Region

region = Region(1, 2, 3)
print(region)  # Region(1, 2, 3)

region.height = -42  # ValueError: Value must be >= 0
```

---

```python
from temperature import Temperature

t = Temperature(25)
print(t.celsius)  # 25

t.celsius = -300  # ValueError: Value must be >= -273.15
```

</div>
