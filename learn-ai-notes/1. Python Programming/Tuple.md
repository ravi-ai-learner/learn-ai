
#python #data-structures #sequences

> [!abstract] Overview
> - A **tuple** is an ordered, immutable sequence of elements in Python.
> - Tuples are defined using parentheses: `my_tuple = (1, 2, 3)`.
> - Elements can be of any data type, and tuples can be nested.
> - Because they are immutable, you cannot add, remove, or change items after creation.
> - Tuples support indexing, slicing, and iteration similar to lists.
> - Used for representing fixed collections, such as the coordinates of a point or the days of the week.
> - Like lists and dictionaries, tuples support mixed types and nesting, but they don't grow and shrink because they are immutable.

```python
> T = (1, 2, 3, 4)

# Indexing
> T[0]
1

# Slicing
> T[1:]
(2, 3, 4)

# Immutable
> T[0] = 2
...error text omitted...
TypeError: 'tuple' object does not support item assignment
```

> [!question] If tuple is immutable, how does the following concatenation worked?
> ```python
> > T + (5, 6)
> (1, 2, 3, 4, 5, 6)
> ```
> The `+` operator does not modify `T`, instead, it creates a new tuple by concatenating the contents of `T` with `(5, 6)`. If you check `T` again:
> ```python
> > print(T)
> (1, 2, 3, 4)
> ```

