# How To Open a file securely


## The Challenge

Reading a file is one of the most common operations performed in many Python programs.

However, reading files can introduce security issues if not handled carefully.

The goal should be to use a stable method across different operating systems that is resistant to malicious input. In Python, both `open()` and `pathlib` are standard tools, but both can leave the door open to path traversal attacks if user input is not strictly controlled.



## The Threat

The primary security risk when reading files is a Path Traversal (also known as Directory Traversal) attack. If an attacker provides a path such as `../../etc/shadow` or `../../root/.bash_history`, they can “climb” out of the intended directory and access sensitive files.

Furthermore, relying on the system’s default encoding (as the first example does) can lead to crashes or data corruption when code is moved from a Linux server to a Windows environment. This can create a denial-of-service (DoS) risk or cause unexpected behaviour.


## Vulnerable Code Example

```python
# Vulnerable: No path validation + unpredictable encoding
def get_file(path):
    with open(path, "r") as f:
        return f.read()
```


## Secure Mitigation

To read files more securely, you should resolve the path to its absolute form and verify that it still resides within a designated “safe” directory.



```python
from pathlib import Path

def get_file_safe(user_input: str, safe_directory: str = "data_folder") -> str:
    base = Path(safe_directory).resolve()
    # Join and resolve the path to remove all '..'
    target = (base / user_input).resolve()

    # The Security Check
    if not target.is_relative_to(base):
        raise PermissionError("Access Denied: Path traversal detected.")
    
    if not target.is_file():
        raise FileNotFoundError("Target is not a valid file.")

    return target.read_text(encoding="utf-8")

```

## Discussion

Neither `open()` nor `pathlib` automatically “jails” execution to a specific directory.

Overview:

| Feature               | open(path)                     | Path(path).read_text()           |
|-----------------------|--------------------------------|----------------------------------|
| Path Traversal Risk   | High                           | High                             |
| Encoding Handling     | Implicit (OS-dependent)        | **Explicit (secure default)**        |
| Logic Errors          | **Easy to forget `f.close()`**     | Handles closing automatically   |
| Input Sanitisation    | None                           | None                             |



`pathlib.Path` is generally considered more secure in a broad sense because it encourages better coding practices that reduce accidental errors—though not solely because of path security. Using `pathlib` is the **modern approach** to file handling in Python.

Key advantages include:

- **Type validation**
By using Path(path), the code ensures the input behaves like a filesystem path. While this is not a sandbox, it makes it easier to use methods such as .resolve() to verify that a path is actually where it is expected to be.

- **Explicit encoding**
By specifying `encoding="utf-8"`, you ensure the file is interpreted consistently regardless of the server’s locale. Relying on the operating system’s default encoding is a common source of “it works on my machine” bugs.

- **Resource management**
`Path.read_text()` is a high-level method that internally handles opening and—crucially—closing the file immediately.

- **Object-oriented logic**
Using the Path object allows you to perform checks such as `.is_file()` or `.exists()` more cleanly than with the older `os.path` module.


:::{tip} 
Use `pathlib` when reading files, but always include a check to ensure the resolved path has not escaped your intended directory. Secure file access requires both safe APIs and explicit validation.
:::