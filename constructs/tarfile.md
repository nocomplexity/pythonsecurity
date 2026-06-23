# TarFile Statement

The Python `tarfile` module makes it possible to read and write tar archives.

Using these methods in Python code can give serious security concerns.


## Security concerns


:::{danger} The default rule is:
Assume all input is malicious.
:::

The `tarfile` module enables reading and writing tar archives. However, `TarFile.extract()` and `TarFile.extractall()` are **inherently dangerous** when handling untrusted archives.

Key risks include:

- **Path traversal** (CWE-22): Malicious members can use absolute paths (`/etc/shadow`), `..` sequences, or symlinks to escape the target directory.
- **Privilege escalation**: Running with elevated privileges allows overwriting system files, SSH keys, or configuration.
- **Sandbox escape**: Breaks out of containers, chroots, or temporary directories.
- **Metadata tampering**: Timestamps and other fields can obscure attacker activity and hinder forensics.
- **Denial of Service**: Historical vulnerabilities (infinite loops in parsing) enabled resource exhaustion.

Tar archives are extremely permissive by design. Malicious tarballs are trivial to craft, and the default extraction behavior offers little protection.

:::{danger}
Using `TarFile.extractall` or `TarFile.extract` is **always dangerous** with untrusted input. Strong mitigations **must** be present.
:::

## Preventive measures


- **Never** extract archives from untrusted sources without inspection and validation.
- Always specify the `filter` argument (Python 3.12+ strongly recommended):

```python
import tarfile

with tarfile.open("archive.tar.gz") as tar:
    tar.extractall(path="/safe/dir", filter="data")  # Safest built-in filter
```

- `filter='data'` strips the most dangerous behaviors (absolute paths, `..`, dangerous symlinks, etc.).
- Avoid `filter=None` (old default) or `filter='tar'` for untrusted data.
- Run extraction inside a **dedicated low-privilege container or VM**.
- Manually validate members before extraction (reject absolute paths, symlinks to outside targets, devices, etc.).
- Consider safer formats when possible (`zipfile` with validation, or libraries enforcing stricter safety).

```{note}
Detection of this construct **always requires human review**. No automated or AI-based analysis can fully assess whether the surrounding context and mitigations are sufficient.
```

**Example:**  

```python
import tarfile

# Vulnerable - classic insecure pattern
with tarfile.open("untrusted.tar.gz") as tar:
    tar.extractall("/tmp/extract")   # Can write anywhere!

# Still risky without proper filter
with tarfile.open("untrusted.tar.gz") as tar:
    tar.extractall(path="/tmp/extract", filter=None)
```

## Discussion

`tarfile` extraction should trigger in-depth security review in every code audit. Defense-in-depth is essential: combine safe filters, privilege dropping, sandboxing, and (when possible) cryptographic verification of archives before extraction. Even with mitigations, handling untrusted tarballs remains one of the riskiest operations in Python.

## More Information


- [Python Documentation — tarfile Extraction filters](https://docs.python.org/3/library/tarfile.html#tarfile-extraction-filter)
- [CVE-2025-4330](https://www.cve.org/CVERecord?id=CVE-2025-4330)
- [CVE-2024-12718](https://nvd.nist.gov/vuln/detail/CVE-2024-12718)
- [CWE-22: Improper Limitation of a Pathname to a Restricted Directory ('Path Traversal')](https://cwe.mitre.org/data/definitions/22.html)
- [CWE-835: Loop with Unreachable Exit Condition ('Infinite Loop')](https://cwe.mitre.org/data/definitions/835.html)
- [What Is The Tarfile Vulnerability in Python?](https://www.securitycompass.com/kontra/what-is-the-tarfile-vulnerability-in-python/)
- [Tarfile: Exploiting the World With a 15-Year-Old Vulnerability](https://www.trellix.com/blogs/research/tarfile-exploiting-the-world/)
- [Summary of Python tarfile Infinite Loop Vulnerability (CVE-2025-8194)](https://zeropath.com/blog/cve-2025-8194-python-tarfile-infinite-loop)


