---
date: 2025-08-01
tags:
  - python
  - sequences
  - built-in-types
  - mutable
---

> [!tip]+ Key Points
> - Sequences in Python include: `str`, `list`, `tuple`, `bytes`, `bytearray`, and `range`.
> - Immutable sequences: `str`, `tuple`, `bytes`, `range`
> - Mutable sequences: `list`, `bytearray`
> - Mutability affects whether an object can be changed in place or not.
> - Use mutable sequences when in-place changes are needed for performance or design reasons.

> [!code]+ Code Snippets
> ```python
> # Immutable sequence
> s = "hello"
> # s[0] = "H"  # Error: strings are immutable
> 
> # Mutable sequence
> l = [1, 2, 3]
> l[0] = 100  # This works
> print(l)  # Output: [100, 2, 3]
> ```

> [!faq]+ FAQs
> **Q**: Why are some sequences immutable in Python?  
> **A**: Immutable types like strings and tuples ensure that their contents can't be altered, which makes them hashable and safe to use as dictionary keys or in sets.

> [!resources]+
> 1. Chapter 6 - Learning Python by Mark Lutz

> [!summary]+ Summary
> - Python sequences can be either mutable or immutable.
> - Knowing which type is which helps prevent unintended side effects or errors.
