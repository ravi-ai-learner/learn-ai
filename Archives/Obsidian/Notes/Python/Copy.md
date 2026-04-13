---
date: 2025-08-04
tags: [python, copy, deepcopy, memory, references, variables, object-model]
---

> [!tip]+ Key Points
> - In Python, **variables reference objects**, not other variables.
> - A **reference** is a pointer from a variable name to an object in memory.
> - All Python objects have two internal headers:
>   - A **type designator**
>   - A **reference counter**
> - Python uses **garbage collection** to reclaim memory for objects no longer referenced.
> - `copy.copy()` makes a **shallow copy** — new outer object, shared inner objects.
> - `copy.deepcopy()` makes a **deep copy** — new outer and inner objects.
> - Integers from `-5 to 256` are **interned** by Python, meaning reused across assignments.

> [!code]+ Code Snippets
> ```python
> import copy
> 
> a = [[1, 2], [3, 4]]
> shallow = copy.copy(a)
> deep = copy.deepcopy(a)
> 
> a[0][0] = 100
> print(shallow)  # [[100, 2], [3, 4]]
> print(deep)     # [[1, 2], [3, 4]]
> 
> # Reference example
> x = 3
> y = 3
> print(x is y)  # True (both reference same interned int object)
> ```

> [!faq]+ FAQs
> **Q**: How many objects are created when I write `x = 3; y = 3`?  
> **A**: Just one object. For small integers (-5 to 256), Python uses **interning**, so both variables reference the **same** shared object in memory.
> **Q**: What does shallow vs deep copy mean for nested objects?  
> **A**: Shallow copies reuse inner objects; deep copies recreate them, so changes to inner structures won’t affect the deep copy.

> [!resources]+
> 1. Chapter 6 - Learning Python by Mark Lutz
> 2. [copy — Shallow and deep copy operations (Python docs)](https://docs.python.org/3/library/copy.html)
> 3. [Python Object Model and Memory Management](https://docs.python.org/3/reference/datamodel.html)

> [!summary]+ Summary
> - Variables point to objects; not to other variables.
> - Python uses references, reference counting, and garbage collection.
> - Copying involves understanding object references: shallow vs deep.
> - Memory optimizations like interning affect behavior in subtle but important ways. 
