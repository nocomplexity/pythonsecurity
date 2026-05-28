---
title: Python in Isolated Mode
short_title: Isolated Mode
---


## Running Python in Isolated Mode (`python -I`)

One of the most effective built-in security features of Python is **isolated mode**, activated using the `-I` command-line flag.

## What does `python -I` do?

When Python is started with the `-I` flag, it runs in **isolated mode**. This mode significantly restricts the environment in which Python executes:

- Ignores all `PYTHON*` environment variables (such as `PYTHONPATH`, `PYTHONHOME`, and `PYTHONSTARTUP`).
- Prevents the automatic addition of the user site-packages directory (`~/.local/lib/pythonX.Y/site-packages` on Unix).
- Limits the modification of `sys.path` to improve predictability and control.
- When running scripts (non-interactive mode), the current directory (`.`) is **not** automatically added to `sys.path`.

Isolated mode is often combined with `-S` (no site) for even stronger isolation:

```bash
python -I -S script.py
```

## When to Use Isolated Mode

| Scenario                                | Security / Reliability Benefit |
|-----------------------------------------|--------------------------------|
| **High-Security Environments**          | Prevents malicious or unintended modifications via environment variables. |
| **Running Untrusted Code**              | Reduces the risk of code being influenced by external PYTHONPATH settings or user-installed packages. |
| **Security Tooling & Scanners**         | Ensures consistent, reproducible behaviour when running SAST, SCA, or custom security scripts. |
| **CI/CD Pipelines**                     | Eliminates “it works on my machine” issues caused by local environment variables. |
| **Auditing and Forensics**              | Provides a clean baseline to analyse whether behaviour stems from the code or the environment. |
| **Sandboxing**                          | Helps contain potentially dangerous scripts by limiting their access to the broader system configuration. |

## Security Value

Running Python with `-I` (and especially `-I -S`) is a strong defensive technique that reduces the attack surface by:

- Preventing attackers from abusing environment variables to inject malicious paths or code.
- Minimising the influence of user-specific or system-wide package installations.
- Making execution more deterministic and auditable.

This is particularly valuable when processing untrusted input, analysing suspicious scripts, or operating in zero-trust environments.

## Recommended Usage

For maximum security and isolation, use the following command:

```bash
python -I -S -X dev -W error script.py
```

- `-I` — Isolated mode
- `-S` — Skip importing the `site` module
- `-X dev` — Enable development mode (extra runtime checks)
- `-W error` — Treat all warnings as errors

> While isolated mode is excellent for security tools and analysis, it is generally **not suitable** for normal application runtime, as many dependencies rely on standard environment behaviour.
