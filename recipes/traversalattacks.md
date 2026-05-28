# How To Prevent Directory Traversal Attacks

## The Challenge 

Directory Traversal is a common but dangerous vulnerability that can affect applications when file paths are built using untrusted user input, or when file paths are used with unchecked input. Python applications are not immune to directory traversal attacks.

In a typical attack, an attacker manipulates path components, such as `../`, to access files that should never be exposed. For example, instead of accessing a permitted file, an attacker could request sensitive system files. When Python applications fail to properly validate or restrict file paths, these malicious inputs are processed as legitimate requests.


## The Threat

The impact of directory traversal can be severe. An attacker could embed or retrieve files on the server through a browser, exposing sensitive configuration files, credentials, or application secrets. In more serious cases, this exposure can potentially lead to remote code execution, especially if writeable directories or executable files are involved.

Directory traversal occurs whenever an application accesses a file without enforcing strict path validation or without ensuring the integrity of the file. To prevent this issue, Python applications should always validate user input, restrict file access to specific directories, and use secure path handling techniques to ensure that user-supplied paths cannot escape intended boundaries. Furthermore, all files read by the programme should be trusted; if this is not possible, a file should be treated as ‘insecure’.

## Vulnerable Code Example

In the following example, the function directly concatenates user-controlled input into a filesystem path without validation.

```python
def save_file(filename, data):

# DANGER: Directly joining user input with a path

with open(”/var/www/data/” + filename, “wb”) as f:

f.write(data)
```

**Why this is vulnerable?**
An attacker can supply a filename such as `../../app.py`. This causes the application to resolve the path as `/var/www/data/../../app.py`, which effectively points to `/var/app.py`. If the attacker overwrites application files or executable scripts, it can lead to total system compromise.


## Secure Mitigation

To prevent this issue, Python applications should always validate user input and use secure path handling techniques to ensure that user-supplied paths cannot escape intended boundaries.


While `os.path.realpath()` is a valid approach, the modern standard in Python is using the `pathlib` module. It provides a more robust way to resolve paths and verify their integrity.

```python
from pathlib import Path

# Resolve the base directory to an absolute, canonical path
BASE_DIR = Path("/var/www/data").resolve()

def save_file(filename, data):
    # Combine and resolve the path to remove symbols like '../' and symlinks
    target_path = (BASE_DIR / filename).resolve()

    # Enforce directory boundary: ensure the target is inside BASE_DIR
    if BASE_DIR not in target_path.parents and target_path != BASE_DIR:
        raise ValueError("Security Alert: Attempted directory traversal")

    with open(target_path, "wb") as f:
        f.write(data)
```


There are of course more mitigations possible. If possible the best approach is always to completely eliminate directory traversal possibilities by stripping all path components from user input.

## Discussion


Understanding and mitigating directory traversal vulnerabilities is critical for building secure Python applications and protecting information and users.

100% automated detection of directory traversal vulnerabilities in Python programs is almost impossible. However, with these simple measures, detection, and prevention are possible, helping to avoid serious security issues when using Python programs developed by others:

* Always create a security design based on a threat model.

* Use a [reliable security checklist before using any Python program](https://nocomplexity.com/checklist-using-python/).

When developing Python applications yourself, the following measures to prevent directory traversal are mandatory:

* Practice [security-by-design](https://nocomplexity.com/documents/securitybydesign/intro.html) principles by creating a secure architecture before writing any code. This also applies when using AI-assisted coding tools.

*  Follow [Python secure programming principles](https://nocomplexity.com/documents/codeaudit/securecoding.html).

*  Always validate your code before release using a trusted open-source security code analyser, such as [Python Code Audit](https://github.com/nocomplexity/codeaudit).


Directory traversal occurs whenever an application accesses a file without enforcing strict path validation or without ensuring the integrity of the file. Furthermore, all files read by the programme should be trusted; if this is not possible, a file should be treated as ‘insecure’.

Understanding and mitigating directory traversal vulnerabilities is critical for building secure Python applications and protecting information and users.


Failing to enforce these protections can expose sensitive files and lead to severe security issues, including remote code execution.