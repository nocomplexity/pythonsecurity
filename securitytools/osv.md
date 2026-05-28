---
title: Vulnerability Scanner
short_title: Vulnerability Scanner
---

**Use OSV-Scanner to identify known vulnerabilities in your Python project’s dependencies.**

Large Python projects frequently include dependencies written in other languages. A comprehensive vulnerability scanner like **OSV-Scanner** is therefore essential for catching issues across your entire dependency tree.

OSV-Scanner is Google’s official open-source command-line tool and frontend for the [OSV.dev](https://osv.dev/) vulnerability database. It intelligently connects your project’s dependencies (both direct and transitive) with known vulnerabilities that actually affect them.

## Getting Started

You can explore for vulnerabilities in Python modules using the OSV database online here:

{button}`OSV-Scanner(online)  <https://osv.dev/list>`

For day-to-day use in projects, install the **OSV-Scanner CLI** (recommended for Python security workflows).


## Why OSV-Scanner?

- It provides **precise matching** using rich vulnerability data from multiple ecosystems.
- It focuses on vulnerabilities that pose a **genuine risk** to your application, helping you avoid alert fatigue.
- It supports a very wide range of languages, package managers, containers, and operating systems.

## Supported Features

OSV-Scanner supports a broad range of technologies, including:

- **Languages**: C/C++, Dart, Elixir, Go, Java, JavaScript, PHP, Python, R, Ruby, Rust (and more).
- **Package Managers**: pip (`requirements.txt`), Poetry, PDM, uv, npm, yarn, Maven, Go modules, Cargo, Gem, Composer, NuGet, and many others.
- **Operating Systems**: Vulnerabilities in OS-level packages on Linux distributions (Debian, Ubuntu, Alpine, etc.).
- **Containers**: Scans container images for vulnerabilities in base images and installed packages.
- **Guided Remediation** (OSV-Scanner v2+): Intelligent upgrade recommendations based on dependency depth, severity, fix availability, and estimated return on investment.

:::{warning}
Checking against known vulnerabilities is **not** as strong as doing a [SAST scan on code](codeaudit). 
:::
