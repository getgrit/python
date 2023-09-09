---
title: Apply common lints to Python
---

Finds bad code patterns and refactors them to a better pattern. The pattens it formats:

1. Replace With Ternary Operation: Replaces an assignment to the same variable done across an if-else with a ternary operator when both are equivalent.

2. Rename Builtin Shadow: Renames variables that shadow builtin variables/functions (i.e. list, getattr, type). - DISABLED

3. Aware DateTime for UTC: To get the current time in UTC use an aware datetime object with the timezone explicitly set to UTC.

4. Binary Operation Identity: Some binary operations can be simplified into constants, this lint performs those simplifications.

5. Boolean If Expression Identity: When a boolean expression is used in an if-else to get a boolean value. Use the boolean value directly.

6. Collection Builtin To Comprehension: Use list, set or dictionary comprehensions directly instead of calling `list()`, `dict()` or `set()`.

7. Collection To Bool: Replace constant collection with boolean in boolean contexts.

8. Comprehension To Generator: Replace unneeded comprehension with generator.

9. Convert Any to In: Converts `any()` functions to simpler `in` statements.

10. Delete Comprehension: Replaces cases where deletions are made via `for` loops with comprehensions.

11. Dictionary Comprehension: Replaces dictionaries created with `for` loops with dictionary comprehensions

```grit
engine marzano(0.1)
language python

// Helper to check if a token is a const literal
pattern is_const_literal() {
    r"(?:\d+.\d+)|(?:\d+)|(?:'.*')|(?:\".*\")$"
}

// Replace With Ternary Operation
pattern replace_with_ternary_op() {
    `
    if $cond:
        $if_body
    else:
        $else_body
    ` as $cond_body where {
        $if_body <: contains `$var = $if_value` && $else_body <: contains `$var = $else_value` where and {
            $if_value <: is_const_literal(),
            $else_value <: is_const_literal(),
            $cond_body => `$var = $if_value if $cond else $else_value`,
        },
    }
}

// Helper to check for builtin's
pattern builtin() {
    or {
        `abs`,
        `aiter`,
        `all`,
        `anext`,
        `any`,
        `ascii`,
        `bin`,
        `bool`,
        `breakpoint`,
        `bytearray`,
        `bytes`,
        `callable`,
        `chr`,
        `classmethod`,
        `compile`,
        `complex`,
        `delattr`,
        `dict`,
        `dir`,
        `divmod`,
        `enumerate`,
        `eval`,
        `exec`,
        `filter`,
        `float`,
        `format`,
        `frozenset`,
        `getattr`,
        `globals`,
        `hasattr`,
        `hash`,
        `help`,
        `hex`,
        `id`,
        `input`,
        `int`,
        `isinstance`,
        `issubclass`,
        `iter`,
        `len`,
        `list`,
        `locals`,
        `map`,
        `max`,
        `memoryview`,
        `min`,
        `next`,
        `object`,
        `oct`,
        `open`,
        `ord`,
        `pow`,
        `print`,
        `property`,
        `range`,
        `repr`,
        `reversed`,
        `round`,
        `set`,
        `setattr`,
        `slice`,
        `sorted`,
        `staticmethod`,
        `str`,
        `sum`,
        `super`,
        `tuple`,
        `type`,
        `vars`,
        `zip`,
        `__import__`,
    }
}

// Rename Builtin Shadow
pattern replace_builtin_shadow() {
    `$after_var` where {
        $after_var <: after `$var = $_` where $var <: builtin() => `my_$var` where {
            $after_var <: maybe contains $var => `my_$var`
        }
    }
}

// Aware DateTime for UTC
pattern replace_utcnow_with_aware_date_time() {
    $new_import = `timezone`,
    `datetime.utcnow()` => `datetime.now($new_import.utc)` where {
        $new_import <: ensure_import_from(source = `datetime`),
    }
}

// Binary Operation Identity
pattern binary_op_identity() {
    or {
        `$var | $var` => `$var`,
        `$var & $var` => `$var`,
        `$var ^ $var` => `0`,
        `$var - $var` => `0`,
        `$var % $var` => `0`,
        `$var / $var` => `1`,
        `$var // $var` => `1`,
    }
}

// Boolean If Expression Identity
pattern boolean_if_expr_ident() {
    // IMPROVEMENT: Could be more intelligent here and figure out if the expression is
    // boolean in itself and therefore does not need the bool() wrapper
    `True if $expr else False` => `bool($expr)`
}

// Collection Builtin To Comprehension
pattern collection_builtin_to_comp() {
    or {
        `list($expr for $x in $arr)` => `[$expr for $x in $arr]`,
        `set($expr for $x in $arr)` => `{$expr for $x in $arr}`,
        `dict(($key, $expr) for $x in $arr)` => `{$key: $expr for $x in $arr}`,
    }

}

// Collection To Bool
pattern collection_to_bool() {
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
}

// Helper to check for functions that accept generators
pattern accept_generator() {
    or {
        `all`,
        `any`,
        `enumerate`,
        `frozenset`,
        `list`,
        `max`,
        `min`,
        `set`,
        `sum`,
        `tuple`,
    }
}

// Comprehension To Generator
pattern comprehension_to_generator() {
    `$func([$expr for $x in $arr])` => `$func($expr for $x in $arr)`
}

// Convert Any to In
pattern convert_any_to_in() {
    `any($x == $val for $x in $arr)` => `$val in $arr`
}

// Delete Comprehension
pattern del_comprehension() {
    `
    for $key in $dict.copy():
        if $key not in $collection:
            del $dict[$key]
    ` => `$dict = {$key: value for $key, value in $dict.items() if $key in $collection}`
}

// Dictionary Comprehension
pattern dict_comprehension() {
    `$val = {}` as $decl where {
        $program <: contains `
            for $key in $iter:
                $var[$key] = $expr
            ` => `$var = {$key: $expr for $key in $iter}`,
        $decl => .,
    }
}

sequential {
    before_each_file(),
    maybe replace_with_ternary_op(),
    // maybe replace_builtin_shadow(),
    maybe replace_utcnow_with_aware_date_time(),
    maybe binary_op_identity(),
    maybe boolean_if_expr_ident(),
    maybe collection_builtin_to_comp(),
    maybe collection_to_bool(),
    maybe comprehension_to_generator(),
    maybe convert_any_to_in(),
    maybe del_comprehension(),
    maybe dict_comprehension(),
    after_each_file(),
}

```

# Replace assign across if-else with ternary operator

```python
if condition:
    x = 1
else:
    x = 2

if condition:
    x = 1.0
else:
    x = 2.0

if condition:
    x = "abcd"
else:
    x = "efgh"

if condition:
    y = 10
    x = "abcd {}".format(y)
else:
    x = "efgh"
```

```python
x = 1 if condition else 2

x = 1.0 if condition else 2.0

x = "abcd" if condition else "efgh"

if condition:
    y = 10
    x = "abcd {}".format(y)
else:
    x = "efgh"
```

<!-- # Rename builtin shadow variables

```python
list = [1, 1, 2, 3, 5, 8]
print(list)

if condition:
    dict = 20
    # do_something() # TODO: uncomment this and it should still pass
    print(dict)

dict([1, 2, 3])
```

```python
my_list = [1, 1, 2, 3, 5, 8]
print(my_list)

if condition:
    my_dict = 20
    # do_something() # TODO: uncomment this and it should still pass
    print(my_dict)

dict([1, 2, 3])
``` -->

# Aware date-time for UTC

```python
from datetime import datetime

this_moment_utc = datetime.utcnow()
```

```python

from datetime import datetime, timezone

this_moment_utc = datetime.now(timezone.utc)
```

# Binary operation identity

```python
x | x
x & x
x ^ x
x - x
x / x
x // x
x % x
```

```python
x
x
0
0
1
1
0
```

# Boolean if expression identity

```python
some_var = True if some_boolean_expression else False
```

```python
some_var = bool(some_boolean_expression)
```

# Collection builtin to comprehension

```python
squares = list(x * x for x in y)
squares = set(x * x for x in y)
squares = dict((x, x * x) for x in xs)
```

```python
squares = [x * x for x in y]
squares = {x * x for x in y}
squares = {x: x * x for x in xs}
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

# Comprehension t generator

```python
hat_found = any([is_hat(item) for item in wardrobe])
hat_found = list([is_hat(item) for item in wardrobe])
```

```python
hat_found = any(is_hat(item) for item in wardrobe)
hat_found = list(is_hat(item) for item in wardrobe)
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

# Delete comprehension

```python
x1 = {"a": 1, "b": 2, "c": 3}
for key in x1.copy():  # can't iterate over a variable that changes size
    if key not in x0:
        del x1[key]
```

```python
x1 = {"a": 1, "b": 2, "c": 3}
x1 = {key: value for key, value in x1.items() if key in x0}
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
