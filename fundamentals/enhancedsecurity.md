---
title: Using python -S for Enhanced Security
short_title: Enhanced Security Mode
---


## Using `python -S` for Enhanced Security

One of the lesser-known but valuable security features in Python is the `-S` command-line flag.

## What does `python -S` do?

The `-S` flag instructs Python to **skip importing the `site` module** during startup. 

By default, Python automatically imports the `site` module, which performs several important tasks:
- Adds `site-packages` directories to `sys.path`
- Processes `.pth` files
- Executes `sitecustomize.py` and `usercustomize.py` (if present)

When you use `python -S`, all of these automatic behaviours are disabled, resulting in a cleaner, more isolated Python runtime.

## When to Use `python -S`

| Scenario                              | Benefit                                                                 |
|---------------------------------------|-------------------------------------------------------------------------|
| **High-Security or Sandboxed Execution** | Prevents loading of potentially malicious code from `sitecustomize.py`, `.pth` files, or tampered site-packages. |
| **Running Untrusted Scripts**         | Reduces the risk of supply chain attacks that rely on automatic package loading. |
| **Minimal & Reproducible Environments** | Ensures execution without interference from installed third-party packages. |
| **Auditing and Forensics**            | Helps determine whether unexpected behaviour originates from your code or from the environment. |
| **CI/CD and Automated Tools**         | Creates consistent, isolated runs for scanners, linters, or security tools. |

## Recommended Command

For maximum isolation, combine `-S` with `-I` (isolated mode):

```bash
python -I -S script.py
```

- `-I` (isolated): Ignores `PYTHON*` environment variables and restricts `sys.path`.
- `-S` (no site): Skips the `site` module.

## Security Value

Using `python -S` (especially with `-I`) significantly reduces the attack surface during Python script execution. It is particularly useful when:
- Running tools that process untrusted input.
- Operating in air-gapped or high-security environments.
- Performing security audits where you want to eliminate external influence.

While not suitable for normal application runtime (*as most dependencies would become unavailable!*), it is a powerful option to  analysis scripts, and defensive execution.
