---
title: Replace if-else with ternary operation where appropriate
---

Replaces an assignment to the same variable done across an if-else with a ternary operator when both are equivalent.

```grit
engine marzano(0.1)
language python

// Helper to check if a token is a const literal
pattern is_const_literal() {
    r"(?:\d+.\d+)|(?:\d+)|(?:'.*')|(?:\".*\")$"
}

`
if $cond:
    $if_body
else:
    $else_body
` as $cond_body where {
    or {
        // The if-else only contains the assignments. We can just rewrite that as a single line.
        and {
            $if_body <: `$var = $if_value`,
            $else_body <: `$var = $else_value`,
            $cond_body => `$var = $if_value if $cond else $else_value`,
        },
        // The if-else body contains more than just the assignments. Here we cannot be sure that
        // assignment is not dependent on a variable defined inside the body of the if-else. So we
        // only rewrite it if the assigned values are constant
        and {
            $if_body <: contains bubble($else_body, $cond, $cond_body) `$var = $if_value` where {
                $else_body <: contains `$var = $else_value`,
                $if_value <: is_const_literal(),
                $else_value <: is_const_literal(),
                $cond_body => `$var = $if_value if $cond else $else_value`,
            }
        }
    }
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

if condition:
    y = 10
    x = "abcd"
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

x = "abcd" if condition else "efgh"
```
