# pip-audit: Python Environment Validation

An essential tool to assess your environment for known vulnerable dependencies is vital. A practical Python specific tool for this purpose is `pip-audit`.


`pip-audit` is a tool for scanning Python environments for packages with known vulnerabilities. It uses the [Python Packaging Advisory Database](https://github.com/pypa/advisory-database) via the PyPI JSON API as a source of vulnerability reports. 
([source](https://pypi.org/project/pip-audit/))


`pip-audit` scans installed Python packages and compares them against publicly disclosed vulnerabilities from trusted advisory databases. It helps you identify packages with known security issues so that you can update, replace, or remediate them as part of your security testing process.

## What pip-audit Does

`pip-audit` analyses **dependency trees**, not source code. It identifies **known vulnerabilities** in package versions, but it does not perform static code analysis and does not examine your application logic. 

:::{attention} It is important to understand its limitations:

* It does **not** protect you from malicious packages.
* It does **not** guarantee that all dependency resolutions can be analysed statically.
* It should not be considered a “secure alternative” to installing dependencies.
* It may not detect vulnerabilities in external components (for example, a vulnerable shared library used by a Python package), particularly where there is no clear version linkage in advisory databases.
* It does not guarantee the detection of every possible transitive exposure if that exposure is not formally associated with the Python package version itself.
:::

`pip-audit` is first and foremost an simple and fast test tool for auditing tool for **known vulnerabilities in Python packages.**

:::{warning}
`pip-audit` is **not** a static code analyser. It analyses dependency trees rather than source code and cannot guarantee complete visibility into all dependency resolution scenarios.
:::

---

## Installation

Install `pip-audit` into your Python environment:

```bash
python -m pip install pip-audit
```

---

## Basic Usage

First, activate your virtual environment (recommended for security testing to avoid contaminating your system environment).

Then run:

```bash
pip-audit
```

This will:

* Inspect all installed packages in the current environment
* Identify known vulnerabilities
* Report affected versions
* Provide information about available fixed versions (where applicable)

---

## Auditing a Requirements File

To audit dependencies defined in a requirements file:

```bash
pip-audit -r requirements.txt
```

This checks the resolved dependency set similarly to installing the requirements, but in an isolated context to minimise conflicts with your current environment.


## Using pip-audit in Security Testing

Within a security testing workflow, `pip-audit` should be used:

* During development, to detect vulnerable dependencies early
* In CI/CD pipelines, to prevent introducing known vulnerable packages
* During periodic security reviews of existing projects
* Before release, as part of a security validation checklist

It is best combined with:

* Static code analysis tools
* Dependency pinning and reproducible builds
* Software composition analysis (SCA) processes
* Regular dependency updates

`pip-audit` helps to identify *known* risks in your dependency tree, but it should be part of a broader, defence-in-depth security testing strategy rather than relied upon as a single line of defence.
