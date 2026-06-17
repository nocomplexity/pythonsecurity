# Logging Configuration


## Security Concerns

Using Python's `logging` module can introduce security risks if sensitive information (such as credentials, personal data, or internal system details) is logged without proper sanitization or access controls. Attackers who gain access to log files or outputs can use this information for reconnaissance, privilege escalation, or data exfiltration attacks.

The `logging.config` module provides convenient ways to configure logging from files or dictionaries. For flexibility, it supports converting configuration text into Python objects — including importing callables from user-defined modules and invoking them with parameters supplied in the configuration. 

While powerful, these mechanisms can be abused to execute arbitrary code if the configuration comes from an untrusted source.

:::{important}
Creating pipes and importing `dicts` for configuration can become a security nightmare.
:::

Additionally, portions of the configuration (especially when using `listen()`) are passed through `eval()`. Although the socket is bound to localhost by default, on multi-user systems this can allow a malicious local user to inject and execute arbitrary code in the context of the logging process by connecting to the listening socket and sending a malicious configuration.

## Preventive Measures

- **Never load configuration from untrusted sources**. Thoroughly review and validate any logging configuration files or dictionaries before use.
- Use the `verify` argument with `logging.config.listen()` to ensure only recognised/signed configurations are applied.
- Prefer programmatic configuration (via `dictConfig` with hardcoded, trusted dictionaries) over file-based configuration in production environments.
- Sanitize all data before logging — never log raw user input, secrets, or sensitive payloads.
- Run logging processes with the principle of least privilege.
- Consider using secure alternatives or wrappers that enforce strict validation on configuration inputs.
- Avoid exposing the `listen()` socket in environments where multiple untrusted users share the same machine.

## Example

**Unsafe configuration loading:**

Dangerous — loading from untrusted source:

```python
import logging.config


logging.config.fileConfig('/path/to/untrusted/logging.conf')

# Or using listen without verification
logging.config.listen()
```

**Safer approach:**

Trusted, hardcoded configuration:

```python
import logging.config


LOGGING_CONFIG = {
    'version': 1,
    'disable_existing_loggers': False,
    'formatters': {
        'standard': {
            'format': '%(asctime)s [%(levelname)s] %(name)s: %(message)s'
        }
    },
    'handlers': {
        'default': {
            'level': 'INFO',
            'formatter': 'standard',
            'class': 'logging.StreamHandler'
        }
    },
    'root': {
        'handlers': ['default'],
        'level': 'INFO'
    }
}

logging.config.dictConfig(LOGGING_CONFIG)
```

**Using listen safely:**

Only accept verified configurations:

```python

logging.config.listen(verify=some_verification_function)
```

## Discussion

The convenience offered by Python's logging configuration system is a double-edged sword. The ability to dynamically import and instantiate objects makes the system highly extensible, but it also opens the door to code execution vulnerabilities similar to deserialization attacks in other languages.

The `eval()` usage and socket-based configuration listening are particularly concerning in shared hosting or multi-user scenarios. Even in single-user environments, a compromised configuration file (e.g., via supply-chain attack) could lead to remote code execution.

Modern security best practice favours explicit, code-reviewed configuration over dynamic loading wherever possible.

## More Information

- [Security considerations — Python Logging Configuration (PSF)](https://docs.python.org/3/library/logging.config.html#security-considerations)
