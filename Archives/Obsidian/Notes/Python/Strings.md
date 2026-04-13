---
date: 2025-08-12
tags:
  - python
  - str
  - built-in-types
  - immutable
  - sequences
---

> [!tip]+ Key Points
> - Python strings are categorized as `immutable` `sequences`, meaning that the characters they contain have a left-to-right positional order and that they cannot be changed in place.
> - If the letter r (uppercase or lowercase) appears just before the opening quote of a string, it turns off the escape mechanism. The result is that Python retains your backslashes literally, exactly as you type them. See `Code Snippet 1` .
> - If you wish to turn off a few lines of code and run your script again, simply put three quotes above and below them, like this: See `Code Snippet 2`. Note that this is probably not significant in terms of performance.

> [!code] Code Snippet 1
> ```python
> myfile = open(r'C:\new\text.dat', 'w')
> ```

> [!code] Code Snippet 2
> ```Python
> X = 1
> """
> import os    # Disable this code temporarily
> print(os.getcwd())
> """
> Y = 2
> ```

> [!faq]- FAQs
> **Q**:   
> **A**: 

> [!resources]+
> 1. [Hash Table - GeeksForGeeks](https://www.geeksforgeeks.org/hashing-data-structure/)

> [!summary]+ Summary
> - Summary here 
