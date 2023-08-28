---
title: Import management for Python
---

Grit includes standard patterns for declaratively adding, removing, and updating imports.

```grit
engine marzano(0.1)
language python

/** This is a utility file, you do not need to implement it yourself */

pattern python_import($imports, $source) {
    or {
      import_from_statement(name=$imports, module_name=dotted_name(name=$source)),
      import_statement(name=dotted_name(name=$source))
    }
}

pattern before_each_file_prep_imports() {
    $_ where {
        $GLOBAL_IMPORTED_SOURCES = [],
        $GLOBAL_IMPORTED_NAMES = [],
    }
}

pattern after_each_file_handle_imports() {
  file($body) where $body <: maybe insert_imports()
}


pattern process_one_source($p, $all_imports) {
    [$p, $source] where {
        $imported_names = [],
        $GLOBAL_IMPORTED_NAMES <: some bubble($p, $source, $imported_names, $all_imports) [$p, $name, $source] where {
            $imported_names += $name,
        },
        $joined_imported_names = join(list = $imported_names, separator = ", "),
        if ($p <: module(statements = some python_import($imports, $source))) {
            $imports => `$imports, $joined_imported_names`
        } else {
            $all_imports += `from $source import $joined_imported_names\n`
        }
    }
}

pattern insert_imports() {
    $body where {
        $all_imports = [],
        $GLOBAL_IMPORTED_SOURCES <: some process_one_source($p, $all_imports),
        if ($all_imports <: not []) {
            $p => `$all_imports\n$p`
        } else {
            true
        }
    }
}

pattern imported_from($source) {
    $name where {
        $program <: module($statements),
        $statements <: some bubble($name, $source) python_import($imports, $source) where {
            $imports <: some $name,
        }
    }
}


pattern ensure_import_from($source) {
    $name where {
        if ($name <: not imported_from($source)) {
            if ($GLOBAL_IMPORTED_SOURCES <: not some [$program, $source]) {
                $GLOBAL_IMPORTED_SOURCES += [$program, $source]
            } else {
                true
            },
            if ($GLOBAL_IMPORTED_NAMES <: not some [$program, $name, $source]) {
                $GLOBAL_IMPORTED_NAMES += [$program, $name, $source]
            } else {
                true
            }
        } else {
            true
        }
    }
}

and {
    before_each_file(),
    contains or {
        python_import(source=`pydantic`) => .,
        python_import(source=`fools`) => .,
        `unittest` as $test where {
            $source = `testing`,
            $test <: ensure_import_from($source),
        },
    },
    after_each_file()
}
```

## Base import statement

```python
from typing import List
from pydantic import BaseModel
from pydantic import More
import fools
```

```python
from typing import List



```

## ensure_import_from

```python
import somewhere

unittest.TestCase()
```

```python
from testing import unittest

import somewhere

unittest.TestCase()
```

## Ensure no duplicate imports

```python
import unittest

unittest.TestCase()
```

```python
import unittest

unittest.TestCase()
```
