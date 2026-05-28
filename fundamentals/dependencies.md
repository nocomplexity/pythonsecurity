---
title: Python Software Dependencies and Definitions
short_title: Python Software Dependencies
---


One of the biggest security challenges in modern Python development lies not in your own code, but in the **dependencies** you use.

A typical Python project depends on dozens — sometimes hundreds — of third-party packages through direct and transitive dependencies. This creates a large and often hidden **supply chain** that attackers increasingly target.

## Dependencies Matter for Security

- **Expanded Attack Surface**: Every dependency inherits its own vulnerabilities and risks. A single compromised or vulnerable package can expose your entire application.
- **Supply Chain Attacks**: Malicious packages on PyPI (via typosquatting, dependency confusion, or maintainer takeovers) have become a serious threat. Attackers can deliver backdoors, credential stealers, or remote code execution with one `pip install`.
- **Hidden Vulnerabilities**: Outdated or unmaintained dependencies are among the most common sources of security weaknesses in Python projects.

Effective dependency management — through version pinning, lock files, and regular vulnerability scanning — is a fundamental part of secure Python development. It enables teams to run applications in production with acceptable levels of risk.


## Dependencies in the Python Ecosystem

Dependencies define which external packages a Python project requires to function, develop, or build. They are typically declared in configuration files such as `requirements.txt`, `setup.py`, `setup.cfg`, or the modern standard `pyproject.toml`.

Understanding the different types of dependencies is essential from a security perspective, as vulnerabilities often originate not from your own code, but from the broader software supply chain.

### I. Dependencies Based on Necessity

These categories describe *why* and *when* a package is needed.

| Type                        | Definition                                                                 | Example Scenario |
|-----------------------------|----------------------------------------------------------------------------|------------------|
| **Direct Dependencies**     | Packages explicitly declared in your project's configuration files as necessary for core functionality. | Your project lists `requests` because you need to make HTTP calls. |
| **Transitive (Indirect) Dependencies** | Packages pulled in automatically by your direct dependencies. They form the hidden, deep layers of the software supply chain. | Your project depends on `requests` (direct). `requests` in turn depends on `urllib3` and `charset-normalizer` (transitive). |
| **Development Dependencies** | Packages required only during development, testing, documentation, or maintenance — not needed in production. | Tools such as `pytest`, `black`, `pylint`, or `Sphinx`. |
| **Optional / Extra Dependencies** | Packages needed only when specific optional features are enabled (usually declared using square brackets). | A library might require `numpy` and `pandas` by default, but only install `matplotlib` if the user specifies `[visualization]`. |

### II. Dependencies Based on Execution Phase

These categories focus on *when* in the lifecycle the dependency is required.

| Type                        | Definition                                                                 | Example Scenario |
|-----------------------------|----------------------------------------------------------------------------|------------------|
| **Runtime Dependencies**    | Any package (direct or transitive) required when the application or library is actually running. | `fastapi` and all its transitive dependencies must be present to run a web server. |
| **Build Dependencies**      | Packages needed only to build or package the project (e.g., creating wheels or source distributions). | `Cython` when building a package with C extensions using `setuptools`. |
| **Host / System Dependencies** | Non-Python libraries or tools that must be installed at the operating system level. These cannot be managed by `pip`. | System libraries such as `libjpeg` or `zlib` required by `Pillow` for image processing. |

### III. Resolution and Environment Dependencies

These categories relate to how dependencies are constrained and resolved.

| Type                          | Definition                                                                 | Example Scenario |
|-------------------------------|----------------------------------------------------------------------------|------------------|
| **Abstract Dependencies**     | Dependencies defined by package name and optional version ranges. They do not pin exact versions. | `pandas>=2.0,<3.0` in `pyproject.toml`. |
| **Locked Dependencies**       | Dependencies pinned to exact versions, typically stored in lock files for reproducibility. | The resolved output of Poetry (`poetry.lock`) or `pip-tools`, containing every direct and transitive dependency at a fixed version. |
| **Virtual Environment Dependencies** | The complete set of packages actually installed in an isolated environment. | All packages shown when running `pip freeze` in an activated virtual environment. |

### Dependency Management Hierarchy

The greatest security and stability challenge in Python dependency management stems from **transitive dependencies**. Even a small project can pull in dozens or hundreds of indirect packages, many of which may contain known vulnerabilities or become compromised.

Modern secure Python development therefore moves beyond simple abstract dependencies (as defined in `requirements.txt` or `pyproject.toml`) and instead relies on **lock files** (e.g. `poetry.lock`, `uv.lock`, or `Pipfile.lock`). These files record the exact versions of *all* direct and transitive dependencies, enabling reproducible and auditable builds.

