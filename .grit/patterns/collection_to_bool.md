---
title: Collection To Bool
---

Replace constant collection with boolean in boolean contexts.

```grit
engine marzano(0.1)
language python

or {
    `if $cond:`,
    r"elif (.*):"($cond),
    `while $cond:`,
} where $cond <: or {
    `[$elms]` as $arr where {
        $elms <: or {
            [$_, ...] where $arr => `True`,
            . where $arr => `False`,
        }
    },
    `{}` => `False`,
    `{$_}` => `True`,
    r"\{.*:.*(?:,.*:.*)*\}" => `True`,
    r"\(\)" => `False`,
    r"\(.+\)"=> `True`,
}
```

# Collection to bool

```python
if ["foo", "boo"]:
    baz()
if ["foo"]:
    baz()
if []:
    baz()
if {}:
    baz()
if {1: 1, 2: 2, 3: 3}:
    baz()
if {1, 2, 3}:
    baz()
if (1, 2, 3):
    baz()
if ():
    baz()
```

```python
if True:
    baz()
if True:
    baz()
if False:
    baz()
if False:
    baz()
if True:
    baz()
if True:
    baz()
if True:
    baz()
if False:
    baz()
```
