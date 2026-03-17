## Behind the Magic
### Unlocking Python's Descriptor Protocol

<center>
<br/>Scott Irwin
<p>&nbsp;
<img src="images/BBGEngineering_black.png"
     style="border: none; box-shadow: none; height: 100px"
     alt="Bloomberg Engineering logo"
/>
<br/>
<p>&nbsp;
https://sjirwin.github.io/behind-the-magic-slides/
</center>

------

## About Me

- Bloomberg Engineering
  - Joined in 2014 as Senior Engineer and Team Lead
  - Python educator
  - Python Guild Leader since 2018
    - Former Co-chair

===

# What are Descriptors

===

# Why Write a Descriptor

------

## Slide Tile 1

<div style="display: flex; gap: 20px;">

```python
class Region:
    def __init__(self, height, width, depth):
        self.height = height
        self.width = width
        self.depth = depth

    def __str__(self):
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

------

## Slide Tile 2

<div style="display: flex; gap: 20px;">

```python
class Region:
    def __init__(self, height, width, depth):
        self.height = height
        self.width = width
        self.depth = depth
    def __str__(self):
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

------

## Slide Tile 3

<div style="display: flex; gap: 20px;">

```python
class Region:
    def __init__(self, height, width, depth):
        self.height = height
        self.width = width
        self.depth = depth
    def __str__(self):
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

------

## Slide Tile 4

```python
# attribute.py

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
```

------

## Slide Tile 5

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

    def __str__(self):
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

===

# Attribute Lookup - Basic View

===

# Live Demo

===

# Attribute lookup - Full View

===

# Common and powerful uses of Descriptors

===

# Conclusion
