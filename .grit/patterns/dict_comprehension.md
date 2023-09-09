---
title: Dictionary Comprehension
---

Replaces dictionaries created with `for` loops with dictionary comprehensions.

```grit
engine marzano(0.1)
language python

`$val = {}` as $decl where {
    $program <: contains `
        for $key in $iter:
            $var[$key] = $expr
        ` => `$var = {$key: $expr for $key in $iter}`,
    $decl => .,
}
```

# Dictionary comprehension

```python
cubes = {}
for i in range(100):
    cubes[i] = i**3
```

```python

cubes = {i: i**3 for i in range(100)}
```
