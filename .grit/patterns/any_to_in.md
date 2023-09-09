---
title: Convert Any to In
---

Converts `any()` functions to simpler `in` statements.

```grit
engine marzano(0.1)
language python

`any($x == $val for $x in $arr)` => `$val in $arr`
```

# Convert any to in

```python
def shout_about_bowlers(hats: list[str]) -> None:
    if any(hat == "bowler" for hat in hats):
        shout("I have a bowler hat!")
```

```python
def shout_about_bowlers(hats: list[str]) -> None:
    if "bowler" in hats:
        shout("I have a bowler hat!")
```
