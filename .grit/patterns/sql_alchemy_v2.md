---
title: SQL Alchemy v1 -> v2
---

Convert SQL Alchemy v1 to v2.

tags: #python, #alpha, #hidden

```grit
engine marzano(0.1)
language python

// This pattern is a WIP WORK
// It is not ready for production use

pattern bulk_update() {
    `$x.update($details, $exec_options)` where {
        $x <: contains `$_.query($model)`,
        $new_statement = `update($model)`,
        // add where/filter clauses
        $x <: maybe contains bubble($new_statement) `$_.filter($filters)` where {
            $new_statement += `.where($filters)`
        },

        // adjust values
        $values = [],
        $details <: contains bubble($values) {
            pair($key, $value) where $values += `$key=$value`
        },
        $new_values = join(list=$values, separator=", "),
        $new_statement += `.values($new_values)`,

        // Add execution options
        $new_statement += `.execution_options($exec_options)`

    } => `with Session(engine, future=True) as sess:
    stmt = ($new_statement)

    sess.execute(stmt)`
}

pattern convert_to_subquery() {
    `$var = select($args)` where {
        $program <: contains `select($query)` where {
            $query <: contains $var
        }
    } => `$var = select($args).subquery()`
}

// this pattern can probably be more general
pattern c_to_selected_columns() {
    `$x.where($args)` where { 
        $args <: contains bubble `$a.c.$b` => `$a.selected_columns.$b`
    }
}

file($body) where $body <: any {
    contains bulk_update(),
    contains convert_to_subquery(),
    contains c_to_selected_columns()
}
```

## grit/example.python

Use the [query API](https://docs.sqlalchemy.org/en/20/changelog/migration_14.html#orm-query-is-internally-unified-with-select-update-delete-2-0-style-execution-available)

```python
session.query(User).filter(User.name == "sandy").update({ password: "foobar", other: "thing" }, synchronize_session="fetch")

stmt1 = select(user.c.id, user.c.name)
stmt2 = select(user.c.id, user.c.name)
stmt3 = select(addresses, stmt2).select_from(addresses.join(stmt1))

stmt = select(users)
stmt = stmt.where(stmt.c.name == "foo")
```

```python
with Session(engine, future=True) as sess:
    stmt = (update(User).where(User.name == "sandy").values(password="foobar", other="thing").execution_options(synchronize_session="fetch"))

    sess.execute(stmt)

stmt1 = select(user.c.id, user.c.name)
stmt2 = select(user.c.id, user.c.name).subquery()
stmt3 = select(addresses, stmt2).select_from(addresses.join(stmt1))

stmt = select(users)
stmt = stmt.where(stmt.selected_columns.name == "foo")
```
