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

- Introduction
  - Title
  - About me
  - Why this talk
- Descriptors
  - What is a Descriptor
  - Descriptor Protocol
  - Descriptor Types
  - Descriptors are everywhere
- Attribute lookup: Basic view
  - Lookup Order - Simple View
  - Lookup Order - Expanded View
- _Live coding_ (limit **15 mins**)
  - Basic demo
    - Non-data Descriptor
    - Data Descriptor
      - Without `__set_name__`
      - With `__set_name__`
  - Case study - caching polar coordinates
  - Case study - ValidatedAttribute
- Attribute lookup: Full view
  - Lookup Order - get
  - Lookup Order - set
  - Lookup Order - delete
  - Key Points
- Writing Your Own Descriptors
  - Some Uses for Descriptors
- Wrapping Up
  - How It All Connects
  - Summary
  - Questions

### Timing

- Intro + pre-demo material: ? min
- Live demo: 15 min
- Wrap up: ? min
