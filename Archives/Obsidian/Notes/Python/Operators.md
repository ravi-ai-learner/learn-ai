---
date: 2025-08-05
tags:
  - operators
  - python
---

> [!tip]+ Key Points
> - The **==** operator, tests whether the two referenced objects have the same values; this is the method almost always used for equality checks in Python.
> - The **is** operator, instead tests for object identity—it returns True only if both names point to the exact same object, so it is a much stronger form of equality testing and is rarely applied in most programs.
> - You can always ask Python how many references there are to an object: the **getrefcount** function in the standard sys module returns the object's reference count.

> [!code] Code Snippets
> ```python
> > L = [1, 2, 3]
> > M = L
> 
> > L == M
> True
> 
> > L is M
> True
> 
> > id(L)
> 5019187392
> 
> > id(M)
> 5019187392
> ```

> [!code] Code Snippets
> ```python
> > L = [1, 2, 3]
> > M = [1, 2, 3]
> 
> > L == M
> True
> 
> > L is M
> False
> 
> > id(L)
> 5019187392
> 
> > id(M)
> 5016244032
> 
> Note that this doesn't apply to integer objects between -5 to 256 because small integers and strings are cached and reused, though, **is** tells us they reference the same single object.
> 
> > X = 42
> > Y = 42
> 
> > X == Y
> True
> 
> > X is Y
> True
> ```

> [!code] Code Snippet
> ```Python
> import sys
> sys.getrefcount(1) # 647 pointers to this shared piece of memory
> 647
> ```

> [!faq]- FAQs
> **Q**:   
> **A**: 

> [!resources]+
> 1. Chapter 6 - Learning Python by Mark Lutz

> [!summary]+ Summary
> - Summary here 
