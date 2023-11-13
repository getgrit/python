---
title: Detected a python logger call with a potential hardcoded secret
---

This may lead to secret credentials being exposed. Make sure that the logger is not logging sensitive information.

```grit
engine marzano(0.1)
language python

pattern logger_level_function() {
    or {
        `debug`,
        `info`,
        `warn`,
        `warning`,
        `error`,
        `exception`,
        `critical`,
    }
}

pattern logger_fn() {
    `getLogger`
}

logger_statement($format_string, $sensitive_var) as $log_call where {
    $log_call <: after `$var = logging.$func($string)` => . where {
        $func <: logger_fn(),
        $func <: ensure_import_from(source = `logging`),
    },
    $var.$log_level($format_string) where {
        $log_level <: logger_level_function(),
        $format_string <: r"(?i).*(api.key|secret|credential|token|password).*%s.*",
        $sensitive_var <: identifier(),
    },
} => `$var.$log_level($format_string.replace($sensitive_var, ""))`
```

## Remove sensitive information from logger

```python
import logging

logger = logging.getLogger("some_app")

def some_api_call(foo):
    return

def call_api(api_key):
    try:
        some_api_call(api_key)
    except:
        logger.error("api call using api key %s failed.", api_key)    # exposed api key, remove it from the format string.
```

```python
def call_api(api_key):
    try:
        some_api_call(api_key)
    except:
        logger.error("api call using api key failed.")
```
