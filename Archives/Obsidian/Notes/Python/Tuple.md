---
date: 2025-07-25
tags:
  - python
  - tuple
---

> [!tip]+ Key Points
> - A **tuple** is an ordered, immutable sequence of elements in Python.
> - Elements can be of any data type, and tuples can be nested.
> - Because they are immutable, you cannot add, remove, or change items after creation.
> - Tuples support indexing, slices, and iteration similar to lists.
> - Used for representing fixed collections, such as the coordinates of a point or the days of the week.
> - Like lists and dictionaries, tuples support mixed types and nesting, but they don't grow and shrink because they are immutable.

> [!code]+ Code Snippets
> ```Python
> > T = (1, 2, 3, 4)
> 
> # Indexing
> > T[0]
> 1
> 
> # Slicing
> > T[1:]
> (2, 3, 4)
> 
> # Immutable
> > T[0] = 2
> ...error text omitted...
> TypeError: 'tuple' object does not support item assignment
> ```

> [!faq]- FAQs
> **Q**:   
> **A**: 

> [!resources]+
> 1. [Hash Table - GeeksForGeeks](https://www.geeksforgeeks.org/hashing-data-structure/)

> [!summary]+ Summary
> - Summary here 
