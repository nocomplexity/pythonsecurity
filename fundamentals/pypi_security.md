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

## Security measures on PyPI

PyPI implements a number of controls designed to protect the registry, package publishers, and package consumers. These controls address different parts of the software supply chain and should not be confused with a guarantee that individual packages are safe.

### Distribution hashes

PyPI publishes cryptographic hashes for distribution files. These hashes can be used to verify that a downloaded file is identical to the file associated with that hash on PyPI. PyPI recommends using a cryptographically strong hash such as SHA-256 rather than MD5. ([PyPI][1])

A hash provides **integrity**, but by itself does not establish **trustworthiness**.

For example:

```text
artifact
   |
   +-- SHA-256 --> "this is exactly this file"
```

It does not establish:

```text
artifact
   |
   +-- "this came from a particular Git commit"
   |
   +-- "this was built by the project's official CI system"
   |
   +-- "the maintainer is trustworthy"
   |
   +-- "the code is free of vulnerabilities"
```

These are different security properties.

### HTTPS and the PyPI download hash

PyPI's package downloads are delivered over HTTPS. TLS protects the connection between the client and the PyPI infrastructure and helps prevent an attacker on the network from substituting a different file.

However, HTTPS should not be confused with publisher authentication.

For example, HTTPS can establish that a client is communicating with the legitimate PyPI service. It does not establish that the maintainer who uploaded a particular release is trustworthy, nor that the artifact was produced from a particular source repository.

Similarly, a PyPI SHA-256 hash identifies the contents of a particular distribution. It does not prove that those contents correspond to a particular Git commit.

### `pip` hash-checking mode

`pip` supports an explicit hash-checking mode using `--require-hashes`. In this mode, hashes are supplied by the user, normally through a requirements file, and `pip` verifies downloaded distributions against those expected hashes.

For example:

```text
SomePackage==1.2.3 \
    --hash=sha256:<expected-hash>
```

Hash-checking mode is deliberately strict. All requirements and dependencies must have hashes, and requirements must be pinned to a specific version, URL, or local path. This makes it useful for reproducible and tightly controlled deployments, but it also makes it less convenient for dependency specifications that intentionally allow versions to change. 

An important distinction is that **`pip` does not provide this protection by default**. Normal `pip install` operations do not require the consumer to provide an independently selected hash for every dependency. 

Hash checking therefore addresses a specific threat: unexpected modification or substitution of a distribution. It does **not** determine whether the expected artifact itself is malicious.

---

## Provenance: Trusted Publishing and attestations

Modern PyPI security mechanisms increasingly address the gap between artifact integrity and artifact provenance.

### Trusted Publishing

PyPI Trusted Publishing allows a project to authorize a particular CI/CD identity, such as a GitHub Actions or GitLab workflow, to publish releases. It uses OpenID Connect (OIDC) and short-lived credentials rather than requiring a long-lived PyPI API token to be stored in the CI system. The short-lived token issued during the process expires after a maximum of 15 minutes. 

This reduces the risk associated with stolen or accidentally exposed long-lived publishing credentials.

Trusted Publishing does **not**, however, mean:

> "Everything produced by this workflow is safe."

A compromised repository, malicious contributor, compromised CI workflow, or unsafe build process can still produce a malicious artifact. PyPI explicitly describes Trusted Publishing as a security mechanism for the publishing process rather than an assertion that the published code is trustworthy. 

### PyPI attestations

PyPI also supports attestations based on Sigstore. An attestation can provide verifiable information linking a distribution to the identity used to publish it, such as a particular CI workflow.

This allows a consumer to obtain stronger evidence about **where an artifact came from** than a bare file hash provides. PyPI documents attestations as a way to verify that a particular distribution was published through a Trusted Publisher and to identify that publisher. 

Attestations should nevertheless be interpreted correctly:

> **Provenance is not the same as trustworthiness.**

An attestation can provide evidence that an artifact came from the expected build and publishing process. It does not prove that the source code is free from vulnerabilities or intentionally malicious behaviour.

---

## PyPI's proactive security measures

PyPI also implements security controls intended to protect the registry and its users from abuse. Examples include:

* **Phishing protection:** PyPI detects and warns about certain untrusted domains used to impersonate PyPI.
* **ZIP archive hardening:** PyPI rejects classes of malformed or ambiguous ZIP archives that could otherwise lead to differences in how package contents are interpreted by different tools.
* **Typosquatting detection:** PyPI performs automated detection of potential typosquatting attempts during project creation.
* **Domain-resurrection protection:** PyPI checks for expired domains associated with projects to reduce the risk that an attacker can acquire a previously trusted domain.
* **Spam prevention:** PyPI applies controls to registrations and other activity associated with abuse campaigns.
* **Malware response:** PyPI has a process for receiving and investigating reports of malicious packages and taking appropriate action.

These measures are important layers of defence, but none of them means that every package available on PyPI has been security-reviewed.

For example, in 2025 PyPI reported processing more than 2,000 malware reports. PyPI also published incident reports covering phishing attacks and other supply-chain security events. 

PyPI has additionally hardened its handling of ZIP archives and wheels against parser-confusion attacks. This illustrates an important point: package-registry security is not limited to detecting malicious Python source code; the package formats and the tools that consume them are also part of the attack surface.

---

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

### Summary

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