# Dynamic Import Statements

:::{danger}
Using dynamic imports is a weakness and leads to security vulnerabilities!
:::

Dynamic imports are especially dangerous if you cannot validate upfront what is being imported.

Dynamic imports can be achieved using:
* `__import__`: This built-in function **SHOULD** never be used. It is an advanced, low-level function that is not needed in everyday Python programming.
* `importlib.import_module()`: If dynamic imports are unavoidable, this function is the preferred approach. However, its use must be validated upfront.
* `importlib.util.spec_from_file_location` in combination with `importlib.util.module_from_spec`

## Security concerns

Dynamic imports present a potential security issue, particularly when the module name originates from an untrusted source. Common risky scenarios include:

- Modules fetched from the internet based on user input
- Clever user input constructs in the code that influence module names
- Attackers importing the `os` module and subsequently calling functions to execute system commands

:::{caution}
Allowing dynamic module imports makes it easy for attackers to execute arbitrary code.
:::

### Why `importlib.import_module()` is preferred over `__import__`

| Aspect | `__import__` | `importlib.import_module()` |
|--------|--------------|----------------------------|
| Purpose | Low-level, legacy function | Modern, explicit API |
| Readability | Obscure, looks like a magic method | Clear, self-documenting name |
| Maintainability | Discouraged for general use | Actively maintained by core Python developers |
| Security | No built-in safeguards | Same risks but cleaner interface |

:::{tip}
If your Python code or package truly must use dynamic module imports, use:
```python
import importlib
module = importlib.import_module("module_name")
```
Avoid using `__import__()` entirely.
:::

## Module imports in Python Plugin systems

From a [security-by-design](https://nocomplexity.github.io/securitybydesign/) perspective, utilizing `importlib.util.spec_from_file_location()` alongside `module_from_spec()` to load plugins introduces severe architectural security risks. This mechanism operates at a low level, bypassing Python’s standard, sandboxed import restrictions and allowing the direct execution of arbitrary code from any accessible file path on the disk.

If the file paths or module specifications are influenced by untrusted input—such as user-controlled configuration files or unvalidated directory paths—an attacker can exploit this to achieve Remote Code Execution (RCE) by tricking the application into importing malicious scripts.


Rather than pointing directly to unpredictable locations on disk, modern Python architectures should leverage **namespace packages** or the standard **`importlib.metadata` entry points API**. By registering plugins via package metadata, the application shifts from a dangerous "pull-from-disk" model to a secure, declarative system where only explicitly installed distribution packages can be loaded.




## Mitigations

There is **always** a security risk when `importlib.import_module()` is used. No mitigation eliminates this risk entirely — it can only be reduced.

Possible mitigations:

1. **Use static analysis tools**: Always use a [Python code audit tool](https://nocomplexity.com/codeaudit/) with module scanning options for all modules within a file.

+++

2. **Pre-audit all possible imports** : Check and understand what will be imported and what security risks are involved. You **MUST** never trust that dynamic imports are safe. Most are not!

+++


3. **Review API design** – Check whether your Python program has or truly needs an API to download or import modules dynamically. Often, a static approach can replace the dynamic one.

+++


4. **Seek expert review** – If you do not trust the implementation, call a security expert to help you. See the [sponsor](../sponsors) page for companies that can assist.

## Example

### Insecure Dynamic Import

```python
# INSECURE: Directly importing user-supplied module name
import importlib

user_input = input("Enter module name to load: ")
module = importlib.import_module(user_input)

# If user enters "os", they can now access system functions
# If user enters "subprocess", they can execute shell commands
module.system("rm -rf /")  # Potentially catastrophic
```

**What is wrong with this code?**
- No validation of user input
- No allowlist of permitted modules
- Attacker can import any standard library or installed module
- Attacker can potentially call dangerous functions after import

### The Vulnerable Pattern for a plugin system

```python
# SECURITY RISK: Avoid this low-level dynamic loading pattern
spec = importlib.util.spec_from_file_location(plugin_name, unsafe_file_path)
module = importlib.util.module_from_spec(spec)
spec.loader.exec_module(module)  # Arbitrary code execution occurs here

```


### No Dynamic Imports (Recommended)

The **most secure** approach is to avoid dynamic imports entirely and use static imports with dispatch tables:

```python
# MOST SECURE: No dynamic imports - use static imports with dispatch
import math
import json
import datetime
import re
from typing import Dict, Callable, Any

# Dispatch table mapping names to actual modules
MODULE_DISPATCH: Dict[str, Any] = {
    "math": math,
    "json": json,
    "datetime": datetime,
    "re": re
}

# Dispatch table for specific functions
FUNCTION_DISPATCH: Dict[str, Callable] = {
    "math.sqrt": math.sqrt,
    "math.pi": lambda: math.pi,
    "json.loads": json.loads,
    "json.dumps": json.dumps,
    "datetime.datetime": datetime.datetime,
    "re.search": re.search
}

def get_module(module_name: str) -> Any:
    """Get a module from the dispatch table - no dynamic import needed."""
    if module_name not in MODULE_DISPATCH:
        raise ValueError(f"Module '{module_name}' not available")
    return MODULE_DISPATCH[module_name]

def get_function(function_name: str) -> Callable:
    """Get a function from the dispatch table - no dynamic import needed."""
    if function_name not in FUNCTION_DISPATCH:
        raise ValueError(f"Function '{function_name}' not available")
    return FUNCTION_DISPATCH[function_name]

# Usage
user_input = "math.sqrt"  # From user input
try:
    func = get_function(user_input)
    result = func(25)  # sqrt(25) = 5
    print(f"Result: {result}")
except ValueError as e:
    print(f"Invalid request: {e}")
```

## Summary of Best Practices

| Practice | Recommendation |
|----------|----------------|
| **Avoid dynamic imports** | Prefer static imports with dispatch tables whenever possible |
| **Never use `__import__()`** | This legacy function should never appear in modern Python code |
| **Use allowlists** | If dynamic imports are unavoidable, maintain a strict allowlist of permitted modules |
| **Validate function access** | Do not expose entire modules — restrict access to specific safe functions |
| **Audit all imports** | Use static analysis tools to detect and review all dynamic import usage |

### Security and Python plugin systems


| Method                                       |   Dynamic discovery | Loads arbitrary files | Recommended |
| -------------------------------------------- |  ----------------- | --------------------- | ----------- |
| `importlib.metadata.entry_points()`          |  Yes               | No                    | ⭐⭐⭐⭐⭐       |
| `importlib.import_module()`                  |  Limited           | No                    | ⭐⭐⭐⭐☆       |
| `pkgutil.iter_modules()` + `import_module()` |  Yes               | No                    | ⭐⭐⭐⭐☆       |
| Namespace packages                           |  Yes               | No                    | ⭐⭐⭐⭐☆       |
| `spec_from_file_location()`                  |  Yes               | **Yes**               | ⭐⭐☆☆☆       |
| `SourceFileLoader`                           |  Yes               | **Yes**               | ⭐⭐☆☆☆       |
| `__import__()`                               |  No                | No                    | ⭐⭐⭐☆☆       |



For new applications, the most robust and secure options are:

1. `importlib.metadata.entry_points()` when plugins are installed as Python packages. This leverages Python's packaging ecosystem and avoids arbitrary file loading.
2. `pkgutil.iter_modules()` + `importlib.import_module()` when plugins reside in a known package within your application. This provides automatic discovery while still using Python's standard import machinery.
3. Reserve `importlib.util.spec_from_file_location()` for cases where you genuinely need to load modules from arbitrary file paths (for example, user-provided scripts in a controlled environment). If you use it, ensure the plugin directory is trusted, validate or authenticate plugin files (for example, using signatures or hashes), and avoid passing user-controlled paths directly to the loader.

From a security perspective, `spec_from_file_location()` and related APIs (`module_from_spec()`, `exec_module()`) are **always** worth reviewing as this methods can bypass the normal import mechanism and can execute arbitrary Python files. By contrast, `import_module()`, `pkgutil.iter_modules()`, and `importlib.metadata.entry_points()` generally represent a lower-risk patterns because they work within Python's standard import and packaging systems. 

:::{tip}
Never trust an import system in Python that has the ability to load arbitrary files!
:::


## References

- [Python `importlib.import_module()` documentation](https://docs.python.org/3/library/importlib.html#importlib.import_module)
- [Python `__import__()` documentation (legacy)](https://docs.python.org/3/library/functions.html#import__)
- [Implicit Execution of Arbitrary Code via Automatic `tools.py` Loading ](https://github.com/MervinPraison/PraisonAI/security/advisories/GHSA-2g3w-cpc4-chr4)
- [CVE-2026-40156](https://nvd.nist.gov/vuln/detail/CVE-2026-40156)
- [CWE-94: Improper Control of Generation of Code ('Code Injection')](https://cwe.mitre.org/data/definitions/94.html)
- [CWE-706: Use of Incorrectly-Resolved Name or Reference](https://cwe.mitre.org/data/definitions/706.html)
```
