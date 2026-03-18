# Talk Outline

## As submitted in the proposal

- Introduction (1 min)
- Motivation for this talk (1 min)
- What are Descriptors? (3 min)
- Why write a Descriptor (1 min)
- Attribute lookup - Basic View (2 min)
- Live coding: Basic Descriptor Examples (6 min)
- Live coding: Case Study (8 min)
- Attribute lookup - Full View (1 min)
- Common and powerful uses of Descriptors (1 min)
- Conclusion (1 min)

### Timing

- Intro + pre-demo material: 8 min
- Live demo: 14 min
- Wrap up: 3 min

## Suggested in feedback

- What descriptors are (protocol: `__get__`, `__set__`, `__delete__`)
- Data vs non-data descriptors
- Lookup order
- `__set_name__` for automatic binding
- Functions as descriptors (method binding)
- classmethod/staticmethod as descriptors
- When to use descriptors vs properties vs `__getattribute__`
- Real-world examples (ORMs, validation)

### Used for the slides (WIP)

- Instroduction
  - Title
  - About me
  - Why this talk
- What are descriptors
  - Protocol: `__get__`, `__set__`, `__delete__`
  - Data vs non-data
    - `__set_name__`
- Attribute lookup - Basic view
- _Live coding_ (limit **15 mins**)
  - Basic descriptor examples
    - Simple descriptor
    - Data vs non-data demo
  - Case study - ValidatedAttribute
  - Case study - caching polar coordinates
- Attribute lookup - Full view
  - `__getattribute__`, `__getattr__`, `__setattr__`, `__delattr__`
  - How they interact with descriptors
- Common and powerful uses of descriptors
  - Functions (method binding)
  - classmethod/staticmethod
  - ORMs, validation, computed attributes
- Conclusion

### Timing

- Intro + pre-demo material: ? min
- Live demo: 15 min
- Wrap up: ? min
