# Logger Class

The `Logger` class serves as an entry point for runtime logging to `stderr`. All instances of `Logger` 
share the same internal state, so enabling a particular log domain for a `Logger` in any scope
enables that log domain for all `Logger` instances in the current process.

## Logger()

```python
from bonsai_ai.logger import Logger

log = Logger()

class MySim(bonsai_ai.Simulator):
    def simulate(self, action):
        log.mydomain("My Log Info")


if __name__ == "__main__":
    ...
    log.set_enabled("mydomain")
    ...

```

Return an instance of `Logger` that reflects the shared state of all active Loggers.

## set_enabled(key)

```python
log = bonsai_ai.logger.Logger()
log.set_enabled("foobar")
log.foobar("baz") # logs "baz" to stderr (along with timestamp and domain info)
log.barfoo("zab") # no log, no error
```

Enables the given log domain for all active Loggers.

| Argument  | Description |
| ---       | ---         |
| `key` | A string describing which log domain to enable. |


## set_enable_all(enable_all)

```python
log = bonsai_ai.logger.Logger()
log.set_enable_all(True)
log.foobar("baz") # logs "baz" to stderr (along with timestamp and domain info)
log.barfoo("zab") # logs "zab" to stderr (along with timestamp and domain info)
```

When set, the `Logger` prints any message, regardless of whether the particular domain
was explicitly enabled. Corresponds to the `--verbose` command line flag.

| Argument  | Description |
| ---       | ---         |
| `enable_all` | A bool describing whether verbose logging is enabled. |

