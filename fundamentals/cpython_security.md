---
title: CPython Security 
short_title: CPython Security
---

## Introduction

Python programs are not inherently secure by default. This extends also to the reference Python runtime environment, **CPython**.

The following relevant security questions are covered in this section:

:::{important} **How is security for CPython arranged?**  
:class: dropdown  
Through the Python Security Response Team (PSRT) for operational handling of vulnerabilities, combined with rigorous testing policies for new releases.
:::

:::{important} **What security tests are performed against a CPython release?**  
:class: dropdown
Comprehensive fuzzing tests via OSS-Fuzz, alongside other automated and manual security validations.
:::

:::{important} **How open is the process around the security aspect for CPython?**  
:class: dropdown
Transparent, with policies and processes documented in PEPs and official handbooks.
:::


## What is CPython?

CPython is the reference and most used implementation of the Python programming language. It consists of an interpreter written primarily in C, which compiles Python source code into bytecode before execution by a virtual machine.

As with any complex software project, CPython has had vulnerabilities reported over time. These are tracked publicly via the [National Vulnerability Database (NVD)](https://nvd.nist.gov/) and the Python Security Response Team.

### CVE Statistics 

- Over 199 CVEs have been registered related to Python core components in 2025.
- The number of Python-related CVEs shows a modest upward trend, partly driven by increased scrutiny from researchers exploring Python in conjunction with new AI models.
- Compared to closed-source projects, open-source projects like CPython are more accessible targets for security research due to their transparency. A frequent misconception is that closed-source (proprietary) solutions are inherently more secure than open-source alternatives like CPython. **This is seldom the case.** The "security by obscurity" approach has repeatedly proven fragile.
- The proportion of Python CVEs relative to the total volume of CVEs reported monthly remains **very low**.


*Python records within the US National Vulnerability Database (NVD):*

![CPython CVEs](/images/cpython_cve.png)

:::{note} **CPython can be considered highly secure**
CPython is generally considered highly secure, especially when you take into account its complexity, the size of its codebase, the relatively low number of reported CVEs, and the large community of developers who review and maintain every code change.

From a security perspective, your efforts are better spent validating third-party Python packages. For example, use [Static Application Security Testing (SAST) tools](https://github.com/nocomplexity/codeaudit) to identify vulnerabilities and other security issues in external dependencies.
:::

## Importance of CPython Security for Python Programs

Weaknesses in the CPython interpreter can have broad impact due to its position as the default and most widely deployed Python runtime. Vulnerabilities here can affect millions of users and deployments.

With the rise of software supply chain attacks, understanding risks in the core interpreter is essential. While most security tooling focuses on application-level issues (e.g., dependencies), interpreter-level flaws require different mitigation strategies.

**Recommended practices**:
- Use Static Application Security Testing (SAST) tools tailored for Python.
- Combine with dependency scanning, dynamic testing, and runtime protections (e.g., sandboxing where applicable).
- Stay updated with official Python releases, which include security patches.

## How is Security for CPython Organised?

The security for CPython is governed by the "Core Team". Specific operational security issues are handled by the Python Security Response team(PSRT).

Note that there is overlap in members.

### Core Team

The Python **Core Team** consists of volunteers and sponsored developers responsible for the overall project, including the CPython codebase. As of July 2026, there are approximately 120+ active core developers.

Core developers have commit rights and authority over the project's infrastructure (GitHub organisation, bug tracker, website, etc.). The process for becoming a core developer is outlined in [PEP 13](https://peps.python.org/pep-0013/), though it relies on community consensus rather than purely objective metrics.

### Python Security Response Team (PSRT)

Security for CPython and related tools (such as pip) is coordinated by the **Python Security Response Team (PSRT)**.  There are around 25 members of the PSRT team.

- Membership is open to qualified individuals (typically core developers or trusted community members with security expertise).
- Release managers are automatically members.
- Processes for handling reports are strictly defined and publicly documented.
- Reports should be sent to `security@python.org`.

Detailed governance is covered in [PEP 811](https://peps.python.org/pep-0811/).

## Security Testing for CPython

CPython benefits from continuous fuzzing through the [OSS-Fuzz](https://github.com/google/oss-fuzz) project:

1. **cpython3**: Primary fuzzing targets for the interpreter core. Fuzz targets, seed corpora, and dictionaries are located in `Modules/_xxtestfuzz/`.
2. **python3-libraries**: Fuzzing for standard library modules, hosted in the [python/library-fuzzers](https://github.com/python/library-fuzzers) repository.

Additional security testing includes:
- Static analysis tools.
- Regression test suites with security-focused cases.
- Manual code reviews for sensitive changes.

While fuzzing results are not published, the open-source nature of CPython allows anyone to run their own security audits or contribute new fuzz targets. Instructions for performing fuzzing test with used fuzzing patterns are [available](https://github.com/python/library-fuzzers).

## Transparency and Openness

The CPython security process is highly transparent:
- Vulnerability handling policies are documented.
- Fixes are public after responsible disclosure.
- The codebase is fully open for independent review.

However, some internal triage discussions are private until resolution to prevent exploitation.

More information:
- [Python Security Documentation](https://devguide.python.org/security/)
- [An Empirical Study of Vulnerability Handling Times in CPython](https://arxiv.org/pdf/2411.00447),  Jukka Ruohonen, May 2025, https://arxiv.org/abs/2411.00447.


