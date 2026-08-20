---
title: PyPI Security Aspects
short_title: PyPI Security
---

## Packages on PyPI are not secure by default

The [Python Package Index (PyPI)](https://pypi.org/) is the primary public package repository for Python. It hosts a very large and continuously growing collection of third-party software packages.

A package being available on PyPI does **not** mean that the package has been security-audited or that its content can be trusted. 

There is an important distinction between **package availability**, **artifact integrity**, **artifact provenance**, and **software trustworthiness**:

* **Availability:** the package and release are hosted by PyPI.
* **Integrity:** the file you downloaded is the same file that was published.
* **Provenance:** there is evidence about where and how the published artifact was produced.
* **Trustworthiness:** the package does not contain malicious or otherwise unacceptable behaviour for your particular use case.

PyPI provides some minor mechanisms that improve integrity and provenance, but it cannot guarantee that a package is not malware, has severe weaknesses and should not be installed or used. 

:::{important}
Using a proven [checklist](/guidelines/secure_use.md) prevents security issues before running an unknown or third-party Python program.
:::

## Source repository and the uploaded PyPI artifact

A common mistake is to assume that the source code visible in a public repository is necessarily identical to the code contained in the package installed from PyPI.

For example, a project may have a repository such as GitHub containing a particular commit or release tag. The maintainer may subsequently build a wheel or source distribution and upload that artifact to PyPI. The artifact is what `pip` installs; the repository is not consulted by `pip` during a normal installation.

Consequently, a compromised maintainer account, compromised build environment, malicious build process, or malicious release workflow could potentially result in an artifact containing code that is not present in the corresponding source repository.

This is one reason why **reproducible builds and verifiable build provenance** are important supply-chain security goals.

PyPI has introduced Trusted Publishing and attestations to improve this situation. Trusted Publishing uses OpenID Connect (OIDC) to allow configured CI/CD systems to publish using short-lived credentials rather than long-lived API tokens. PyPI attestations can provide cryptographically verifiable information about the identity used to publish an artifact. These mechanisms increase confidence in the provenance of an artifact, but they do not constitute a security audit of the package or guarantee that its code is benign. 


## Security Risk when installing a package

A Python package can contain arbitrary executable code. Depending on the package and the privileges of the process installing or importing it, malicious code could, for example:

* read files accessible to the current user;
* access environment variables, configuration files, or credentials;
* communicate with external systems;
* exfiltrate sensitive information;
* modify files or application data;
* execute additional programs;
* establish persistence;
* abuse available network or computing resources; or
* interfere with the availability or operation of the system.

These are not properties unique to PyPI. They are inherent consequences of installing and executing third-party software.

A package can also be **unintentionally insecure** without being malicious. It may contain a vulnerability, use an insecure dependency, expose sensitive information through logging, or make assumptions that are unsafe in a particular deployment environment.

Therefore, the appropriate security question is not:

> "Is this package on PyPI?"

but rather:

> "Do I have sufficient reason to trust this particular artifact and its provenance for this particular use?"

:::{tip}
Use only official, managed repositories when installing Python packages. Never ever install Python packages from a location that you can not asses, like some random `https://` location to a Python `whl` or `sdist`.
:::

---

## Security Measures on PyPI

The Python Package Index (PyPI) implements a range of controls designed to defend the registry, package publishers, and consumers against abuse. These measures form important layers of defence across the software supply chain, though they do **not guarantee that individual packages are safe**:

* **Upload Window Restrictions:** New files uploaded to releases older than 14 days are rejected to prevent long-stable releases from being poisoned if project tokens or workflows are compromised.

* **Phishing Protection:** PyPI actively monitors and flags untrusted domains designed to impersonate the index.
* **ZIP Archive Hardening:** Rejects malformed or ambiguous ZIP archives to prevent parser-confusion attacks and ensure consistent interpretation across tools.
* **Typosquatting & Spam Control:** Automated detection identifies potential typosquatting during project creation, alongside controls to block automated registration abuse.
* **Domain-Resurrection Safeguards:** PyPI monitors expired domains linked to active projects to prevent attackers from hijacking previously trusted domains.
* **Malware Response:** Active processes exist to investigate reports of malicious packages and remove them. In 2025 alone, PyPI processed over 2,000 malware reports while publishing detailed incident analyses on phishing and supply-chain events.

### Distribution Hashes & HTTPS

PyPI delivers package downloads over HTTPS, using TLS to secure the connection and prevent network-level tampering. Additionally, PyPI publishes cryptographic hashes (preferring SHA-256 over MD5) to ensure file integrity.

However, integrity and transport security do not equal trustworthiness:

* **HTTPS** proves you are communicating with the official PyPI infrastructure, but does not verify maintainer legitimacy or source origin.
* **SHA-256 Hashes** confirm that a downloaded file matches the registry artifact, but cannot prove it corresponds to a specific Git commit, originated from an official CI system, or is free of vulnerabilities.

### `pip` Hash-Checking Mode

`pip` supports strict verification via the `--require-hashes` flag. When enabled, every requirement and dependency must be pinned to a specific version or path and matched against an explicit, user-supplied hash:

```text
SomePackage==1.2.3 \
    --hash=sha256:<expected-hash>

```

This prevents unexpected modification or substitution of distributions, making it ideal for reproducible deployments. Because normal `pip install` commands do not enforce hash checking by default, consumers must opt in explicitly. This mechanism addresses file substitution, but cannot determine if the expected artifact itself is malicious.

### Provenance: Trusted Publishing & Attestations

Modern PyPI security focuses on bridging the gap between artifact integrity and supply-chain provenance:

* **Trusted Publishing:** Projects can authorise specific CI/CD identities (such as GitHub Actions or GitLab workflows) using OpenID Connect (OIDC). Short-lived credentials expire within 15 minutes, removing the need for long-lived PyPI API tokens.
* **PyPI Attestations:** Built on Sigstore, attestations provide verifiable evidence linking a distribution to its publishing identity and CI workflow.

While these tools prove where an artifact came from and how it was published, provenance is not an assertion of code quality. A compromised repository or build pipeline can still publish malicious artifacts using valid attestations. Provenance verifies origin, not trustworthiness.

:::{warning}
An attestation can provide evidence that an artifact came from the expected build and publishing process. It does not prove that the source code is free from vulnerabilities or intentionally malicious behaviour.
:::



## Security challenges when using PyPI

The principal security challenges in the Python package ecosystem include:

* **Malicious packages:** attackers may deliberately publish packages containing malicious code.
* **Compromised maintainer accounts:** an attacker who gains control of a legitimate project can publish a malicious release under a trusted package name.
* **Compromised build or release infrastructure:** an attacker may modify an artifact during the build or publication process without modifying the source repository.
* **Dependency confusion and package-name attacks:** attackers may exploit package-resolution behavior or publish packages with names resembling legitimate dependencies.
* **Typosquatting:** attackers may register names that are visually or semantically similar to popular packages.
* **Dependency vulnerabilities:** a package may be trustworthy while one of its dependencies contains a vulnerability.
* **Abandoned or compromised projects:** a previously trustworthy project may become unmaintained or change ownership.
* **Build reproducibility:** consumers may lack a practical way to independently reproduce an artifact and compare it with the published distribution.
* **Provenance verification:** consumers need reliable mechanisms to establish how, where, and by whom a distribution was produced.
* **Credential theft:** compromised maintainer accounts, API tokens, or CI/CD credentials can be used to publish malicious releases.

These threats cannot be solved by a single control.

A mature Python software-supply-chain security strategy therefore combines several layers: carefully selecting dependencies, pinning versions where appropriate, reviewing important packages, keeping dependencies updated, using vulnerability information, protecting maintainer and CI/CD credentials, using Trusted Publishing where possible, verifying provenance and attestations where available, and using hash-locked installations for environments that require strict reproducibility.

## Summary

**PyPI provides a distribution mechanism and a number of important security controls; it is not a security certification authority for Python packages.**


A PyPI package should therefore be treated as third-party software. The fact that a package is hosted on PyPI, is downloaded over HTTPS, has a SHA-256 hash, is downloaded millions of times per day, or even has a valid provenance attestation does not, by itself, establish that the package is secure, free from weaknesses, or free from vulnerabilities.


Remember: A weakness is a flaw, design issue, or insecure coding practice that may introduce a security risk; a vulnerability is a weakness that can be exploited to compromise the confidentiality, integrity, or availability of a system in a particular context.



Security depends on what property is being verified:

| Mechanism                  | What it helps establish                                       | What it does not establish                                                 |
| -------------------------- | ------------------------------------------------------------- | -------------------------------------------------------------------------- |
| HTTPS                      | Secure communication with the PyPI infrastructure             | That the package is trustworthy                                            |
| PyPI file hash             | Identity/integrity of a particular distribution               | That the distribution is benign or built from a particular source revision |
| `pip --require-hashes`     | That installed distributions match consumer-specified hashes  | That the expected distributions are safe                                   |
| Trusted Publishing         | Which configured publishing identity was authorized to upload | That the resulting code is safe                                            |
| Attestations               | Cryptographically verifiable artifact provenance              | That the source code or artifact is free of vulnerabilities                |
| Security/malware reporting | Detection and response to known or reported abuse             | That previously unreported packages are safe                               |

The goal is therefore not to establish that **"PyPI is safe"**, but to build enough independent assurance around each dependency that its risk is acceptable for the intended environment.

:::{important}
Before installing a package from PyPI, you SHOULD assess its security and trustworthiness.

A good Python Static Application Security Testing (SAST) identifies weaknesses patterns in Python source code. 

For a quick preliminary assessment, [**Python Code Audit**](https://github.com/nocomplexity/codeaudit) can be used from a web browser to inspect packages available on PyPI without requiring you to install the package locally. 
:::