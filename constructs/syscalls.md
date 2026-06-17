---
title: sys Module:Tracing and Profiling Hooks 
short_title: sys Module
---


The `sys` module provides low-level functions that allow deep introspection and modification of Python's execution behaviour. The functions `sys.settrace()`, `sys.setprofile()`, and `sys.call_tracing()` are particularly powerful but introduce serious security risks when used in untrusted or production environments.

## Security Concerns

These hooks grant near-complete visibility and control over code execution:

- **Information disclosure**: Trace functions receive frame objects containing local and global variables, function arguments, return values, file paths, and line numbers. This can leak credentials, API keys, cryptographic material, or business logic.
- **Malicious monitoring / backdoors**: A compromised or malicious trace/profile function can silently log sensitive data, exfiltrate information, or record user inputs.
- **Control flow manipulation**: Trace functions can modify variables (`frame.f_locals`), alter return values, or influence execution paths — effectively allowing attackers to bypass security checks or inject logic.
- **Denial of Service**: Poorly written or malicious hooks (especially line-level tracing) can cause severe performance degradation, stack overflows, or make the application unresponsive.
- **Stealth and persistence**: These hooks can be installed early (e.g., via sitecustomize, malicious dependencies, or plugins) without obvious signs to the user or developer.
- **Bypassing sandboxes**: In environments attempting to restrict code (REPLs, plugin systems, notebook servers), these functions often provide a way to escape or undermine isolation.

:::{note}
`sys.call_tracing()` is intended for use inside debuggers but still operates within the tracing framework and carries similar risks.
:::


## Preventive Measures

* **Avoid in production**: Do not use `settrace()`, `setprofile()`, or `call_tracing()` in production code unless absolutely required for debugging or monitoring, and only under strict controls.

+++

* **Sandbox untrusted code**: Run user-supplied or untrusted Python code in strong isolation:
  - Operating system containers (Docker, LXC)
  - Virtual machines
  - Restricted Python environments (e.g., with `restrictedpython` or custom `__builtins__`)

+++

* **Disable introspection features**: In REPLs, web-based Python execution, or plugin systems, explicitly remove or block these functions.

+++

* **Use safer alternatives**:
  - For profiling: `cProfile`, `pyinstrument`, or `sys.monitoring` (Python 3.12+)
  - For debugging: Standard debuggers with controlled attachment
  - For auditing: `sys.audit()` hooks (added via C API for security-critical use)

+++

* **Principle of least privilege**: Run applications with minimal permissions and monitor for unexpected tracing activity.


:::{caution}
Avoid using these hooks in production unless absolutely necessary. Always combine with strong sandboxing when executing untrusted code.
:::

## Example

**Legitimate (but risky) debugging usage**:

```python
import sys

def trace_calls(frame, event, arg):
    if event == 'call':
        print(f"Calling: {frame.f_code.co_filename}:{frame.f_lineno} - {frame.f_code.co_name}")
    return trace_calls

# Enable tracing (use with extreme caution)
sys.settrace(trace_calls)

# ... run code ...

sys.settrace(None)  # Disable
```

**Malicious example (what attackers can do)**:

```python
import sys

def malicious_trace(frame, event, arg):
    if event == 'call' and 'password' in str(frame.f_locals):
        # Exfiltrate sensitive data
        print("STOLEN:", frame.f_locals.get('password'))
    # Could also modify frame.f_locals to bypass checks
    return malicious_trace

sys.settrace(malicious_trace)
```

## Discussion

The `settrace` and `setprofile` mechanisms are designed for debuggers and profilers, giving them intentional access to the interpreter's internals. While powerful for development, they are inherently dangerous in any multi-user, plugin-based, or partially trusted environment.

Because these hooks operate at the interpreter level, traditional Python-level defences (such as restricting `__builtins__`) are often insufficient. Malicious code can install hooks that persist across threads or modules.

Modern alternatives like `sys.monitoring` (Python 3.12+) provide more structured and performant tooling, but the core security advice remains: treat tracing and profiling capabilities as privileged operations that should be tightly controlled or disabled.


## More Information

* [Python Documentation](https://docs.python.org/3/library/sys.html)
* [sys.settrace()](https://docs.python.org/3/library/sys.html#sys.settrace)
* [sys.setprofile()](https://docs.python.org/3/library/sys.html#sys.setprofile)
* [PEP 669 – Low Impact Monitoring for CPython](https://peps.python.org/pep-0669/) (introduces `sys.monitoring`)
