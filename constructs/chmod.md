# The `os.chmod` Function

Using Python's `os.chmod()` function to manage file permissions is rarely the right approach. Sooner or later, your program may be shared with or used by others, and hard-coded permission changes can become a significant security liability.

## Security concerns

Most Python programs tend to run with overly broad authorisations. This is particularly problematic when programs are distributed and executed in diverse environments.

When reviewing third-party code, pay special attention to any use of `os.chmod()`.

:::{tip}
Always check whether `chmod` is used in the code, and understand why.
:::

:::{warning}
Use of `chmod` in programs is a recipe for security disasters and privacy breaches.
:::

### Specific risks include:

1. **Overly permissive file access** – Setting permissions such as `0o777` (read, write, execute for everyone) can expose sensitive data to unauthorised users on multi-user systems.

2. **Race conditions (TOCTOU)** – The time between checking a file's permissions and changing them (or using it) can be exploited by an attacker to replace or modify the file.

3. **Portability issues** – Permission models differ between operating systems (for example, Windows does not support Unix-style permission bits). Code using `os.chmod()` may behave unexpectedly or fail entirely on non-POSIX systems.

4. **False sense of security** – Relying on `chmod` to secure a file ignores other attack vectors, such as process memory, temporary copies, or backups with different permissions.

5. **Hard-coded absolute paths** – Using `os.chmod()` with fixed paths can unintentionally affect critical system files if the program is run with elevated privileges.

## Preventive measures

- **Avoid changing permissions programmatically where possible.** Set correct permissions at file creation time using `os.open()` with the `mode` parameter, rather than creating a file and then calling `os.chmod()`.

- **Use the most restrictive permissions necessary.** For example, use `0o600` (read/write for owner only) for sensitive data files, and `0o644` for non-sensitive files that need to be readable by others.

- **Prefer higher-level abstractions.** Consider using the `pathlib` module or `tempfile` for temporary files, which handle permissions more safely by default.

- **Validate before changing.** If you must change permissions, check the current file path and ownership first, and handle errors gracefully.

- **Consider using operating system tools or configuration management** (such as Ansible, Puppet, or systemd service files) to manage permissions declaratively instead of embedding `chmod` calls in Python code.

## Example

### Vulnerable code

```python
import os

# Dangerous: sets world-writable permissions on a sensitive file
with open("secrets.txt", "w") as f:
    f.write("api_key=12345")

os.chmod("secrets.txt", 0o777)  # Anyone can read, write, or delete this file
```

### Safer alternative


```python
from pathlib import Path

# Safer: create file with explicit permissions via pathlib
path = Path("secrets.txt")
path.write_text("api_key=12345")
path.chmod(0o600)  # Still requires chmod, but combined with path object
```

:::{note}
Even with `pathlib`, changing permissions after creation carries some risk. Where possible, design your program to rely on the operating system's default umask or use dedicated secrets management tools (such as environment variables or vault services) instead of file-based permissions.
:::

## More information

- [Python `os.chmod()` documentation](https://docs.python.org/3/library/os.html#os.chmod)
- [CWE-276: Incorrect Default Permissions](https://cwe.mitre.org/data/definitions/276.html)
- [CWE-732: Incorrect Permission Assignment for Critical Resource](https://cwe.mitre.org/data/definitions/732.html)
- [Python `pathlib.Path.chmod()` documentation](https://docs.python.org/3/library/pathlib.html#pathlib.Path.chmod)



