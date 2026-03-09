# Behind the Magic: Unlocking Python's Descriptor Protocol

## Abstract

Ever wondered how @property really works? Or why setting an attribute sometimes feels … magical? That curiosity led me deep into one of Python’s most powerful, yet hidden, features: the **descriptor protocol**.

In this talk, we’ll reveal the magic of how Python decides what happens when you access object attributes. Through simple, live-coded examples, I’ll demonstrate how descriptors enable read-only, cached, and validated attributes - and how they power familiar tools like properties, class methods, and slots.

You’ll walk away with a clearer mental model of Python’s object model, a few new “aha!” moments, and maybe the inspiration to build a descriptor or two yourself.

## Description

Descriptors are one of the most powerful - and most invisible - parts of Python. They’re the foundation for familiar features like `@property`, `@classmethod`, and even `__slots__`. However, despite their importance, many Python developers (my past self included) use them every day without knowing or understanding how they work.

We’ll see some of the magic of descriptors in action as I explore them through live coded demos - starting with simple, focused examples, then moving on to more complete use cases. Along the way, we’ll look at how Python handles attribute lookup, the difference between data and non-data descriptors, and how descriptors can be used for computed attributes, caching, and validation.

The goal is to demystify descriptors and demonstrate that they’re not just for language wizards - once you understand the underlying mechanics, writing your own descriptor is surprisingly straightforward. Attendees will walk away with a deeper understanding of Python’s object model and a new perspective on features they use daily.

At the root of this talk is curiosity. After seeing a lightning talk on descriptors, I couldn’t stop wondering how `@property` actually works under the hood. That curiosity led me down a deep rabbit’s hole of Python internals and ultimately to developing this talk.

## Outline

- Introduction [1 min]
- Motivation for this talk [1 min]
- What are Descriptors? [3 min]
- Why write a Descriptor [1 min]
- Attribute lookup - Basic View [2 min]
- Live coding: Basic Descriptor Examples [6 min]
- Live coding: Case Study [8 min]
- Attribute lookup - Full View [1 min]
- Common and powerful uses of Descriptors [1 min]
- Conclusion [1 min]

## Biography

As a senior engineer at Bloomberg, Scott Irwin has spent more than a decade bridging engineering and education - building Python applications, leading development teams, and teaching hundreds of engineers through Bloomberg’s internal training programs. He’s passionate about helping developers write cleaner, more maintainable code.

Beyond Bloomberg, Scott has taught Python to a broader audience through live online training events on the O’Reilly learning platform.
