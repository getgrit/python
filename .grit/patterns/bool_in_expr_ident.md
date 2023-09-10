---
title: Boolean If Expression Identity
---

When a boolean expression is used in an if-else to get a boolean value, use the boolean value directly.

```grit
engine marzano(0.1)
language python

// IMPROVEMENT: Could be more intelligent here and figure out if the expression is
// boolean in itself and therefore does not need the bool() wrapper
`True if $expr else False` => `bool($expr)`
```

# Boolean if expression identity

```python
some_var = True if some_boolean_expression else False
```

```python
some_var = bool(some_boolean_expression)
```
