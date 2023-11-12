---
title: Detected a python logger call with a potential hardcoded secret
---

This may lead to secret credentials being exposed. Make sure that the logger is not logging sensitive information.

```grit
engine marzano(0.1)
language python

pattern logger_declaration() {
    `$logger = logging.getLogger($logger_name)`
}

pattern logger_function() {
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

pattern logger_call() {
    $logger_func where {
        $logger_func <: logger_function(),
        $logger_func <: is_imported_from(source = `logging`),
    }
}

logger_statement($logger, $logger_func, $format_string, $sensitive_var) as $log_call where {
    $log_call <: `$logger.$logger_func($format_string, $sensitive_var, ...)` => . where {
        $logger <: logger_declaration(),
        $logger_func <: logger_call(),
        $format_string <: r"(?i).*(api.key|secret|credential|token|password).*%s.*",
        $sensitive_var <: identifier()
    }
} => `$logger.$logger_func($format_string.replace($sensitive_var, ""), ...)`
```

## Remove sensitive information from logger

```python
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
