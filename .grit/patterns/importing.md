---
title: Import management for Python
---

Grit includes standard patterns for declaratively adding, removing, and updating imports.

```grit
engine marzano(0.1)
language python

pattern our_import_statement($imports, $source) {
    or {
      import_from_statement(name=$imports, module_name=dotted_name(name=$source)),
      import_statement(name=dotted_name(name=$source))
    }
}

and {
    // before_each_file(),
    contains or {
        our_import_statement(source=`pydantic`) => .,
        our_import_statement(source=`fools`) => .,
        //`unittest` as $test where {
        //  $source = `"unittest"`,
        //  $test <: ensure_import_from($source),
        //},
    },
    // after_each_file()
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
