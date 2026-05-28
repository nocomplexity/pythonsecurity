---
title: Key Definitions and Python Security Terms
short_title: Definitions
---


Clear and consistent terminology is essential in cybersecurity. However, the industry often suffers from hype, overlapping definitions, and vendor-specific jargon. In this book, the following definitions are used for precision and clarity within the Python security context.

## Core Python Packaging Terms

**Import Package**  
An import package is a Python module (or collection of modules) that can be imported using the `import` statement.

**Distribution Package**  
A distribution package is a versioned archive of software that can be installed using tools such as `pip`. It is often synonymous with a “project” on PyPI. Distribution packages typically consist of one or more import packages (modules) bundled together.

> **Security Note**: Every distribution package contains code — either pure Python modules or extensions written in other languages (e.g., C, C++, Rust, or Go). From a security perspective, **all dependencies**, both direct and transitive, should be regularly scanned for vulnerabilities. 

**Python Module**  
A Python module is a single `.py` file (or compiled equivalent) containing Python code that can be imported into other programs.

**Python Library**  
The term “library” has no strict formal definition in Python. It is commonly used interchangeably with “module” or “package”. The **Python Standard Library** is the best-known example of a Python library.

**Python Standard Library (PSL)**  
The Python Standard Library is the collection of modules and packages included with every standard Python installation. It provides broad functionality for common tasks (file I/O, networking, cryptography, data processing, etc.), making Python a “batteries-included” language. While generally robust, certain functions within the PSL can introduce security weaknesses depending on how they are used.

**Standard Python Module**  
A module that is included with the default Python distribution as part of the Python Standard Library. Although these modules undergo extensive review, some APIs can still lead to vulnerabilities (e.g., unsafe use of `eval()`, `pickle`, or XML parsers). Static Application Security Testing (SAST) is recommended to identify and mitigate such risks in your codebase.

## Security Concepts

**Weakness**  
A condition in software, firmware, hardware, or a service that, under certain circumstances, could contribute to the introduction of a vulnerability. A weakness is not necessarily exploitable on its own.

**Vulnerability**  
An exploitable weakness. A vulnerability is a specific flaw that an attacker can leverage to cause harm, such as unauthorised access, data leakage, or remote code execution.

**Threat**  
An external event, actor, or action that has the potential to exploit a vulnerability. It represents the intent or mechanism of harm.

**Advanced Persistent Threat (APT)**  
An Advanced Persistent Threat occurs when an attacker successfully compromises a network, establishes a long-term presence on one or more systems, and extracts data over an extended period. Some APTs eventually reveal themselves through destructive actions (e.g., WannaCry ransomware or Stuxnet), but most aim to remain undetected for as long as possible.

**Static Application Security Testing (SAST)**  
SAST is a white-box security testing methodology that analyses source code, documentation, and design artefacts for potential security weaknesses without executing the program. In Python projects, SAST tools scan code for risky patterns such as unsafe use of `eval()`, insecure deserialisation, or improper input validation.

**Software Composition Analysis (SCA)**  
SCA is the process of identifying and evaluating the security risks associated with open-source and third-party dependencies in a project. While valuable, SCA is limited by the fact that many vulnerabilities in dependencies remain unreported or undiscovered. SCA should therefore be treated as one layer within a broader security strategy, not a complete solution.

