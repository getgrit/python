---
title: Use dict.get with default instead of if-else
---

Join multiple with statements into a single one. Rule [SIM401](https://github.com/MartinThoma/flake8-simplify/issues/72) from [flake8-simplify](https://github.com/MartinThoma/flake8-simplify).

Caveat: if either the key or the default value are functions, they will be called a different
number of times in the generated code. We may need to enforce that `$key` and `$default` don't
call any function.

```grit
engine marzano(0.1)
language python

// NOTE: if $key or $default call functions with side effects, this transform is not safe

`
if $key in $dict:
    $var = $dict[$key]
else:
    $var = $default
` => `$var = $dict.get($key, $default)`
```

# Replace if-else with dict.get()

```python
if "my_key" in example_dict:
    thing = example_dict["my_key"]
else:
    thing = "default_value"

if f() in example_dict:
    thing = example_dict[f()]
else:
    thing = "default_value"


# Left as is

if "name" in d:
    name = d[name]
else:
    name = "foo"

if "name" in d:
    name = d["name"]
else:
    surname = "foo"
```

```python
thing = example_dict.get("my_key", "default_value")

thing = example_dict.get(f(), "default_value")


# Left as is

if "name" in d:
    name = d[name]
else:
    name = "foo"

if "name" in d:
    name = d["name"]
else:
    surname = "foo"
```
