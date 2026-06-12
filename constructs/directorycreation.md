# Directory Creation

Creation of a directory from within a Python program can pose security issues if not handled carefully.

Directory creation is possible with several Python constructs:
* `os.makedirs`
* `os.mkdir`
* `os.mkfifo` 
* `os.mknod`

Direct file system calls require careful input validation to prevent vulnerabilities.

## Security concerns

Creating a directory using Python code is in essence not a security issue. **BUT...**

Minimal needed is validation of input if a directory is created upon user input or based on other input.

The following security risks are present:

### 1. Directory Traversal/Path Manipulation

* **Risk:** If a program takes user-provided input for directory names or paths and directly uses it to create directories without proper sanitisation, an attacker could supply paths like `../../../../etc/malicious_dir` to create directories outside of the intended scope. This could lead to:
    * **Overwriting or interfering with system files/directories:** An attacker might create a directory with the same name as a critical system file, potentially causing the system to malfunction or execute malicious code.
    * **Denial of Service (DoS):** Creating directories in critical system locations or filling up disk space in unexpected places.
    * **Information Disclosure:** If the attacker can control the path where other files are written, they might be able to read or write to sensitive locations.
* **Mitigation:**
    * **Validate and Sanitise Input:** Never trust user input. Always sanitise paths by:
        * **Restricting to a Base Directory:** Use `os.path.abspath()` and `os.path.join()` to construct absolute paths and then check that the resulting path is a sub-path of an allowed base directory.
        * **Normalising Paths:** Use `os.path.normpath()` to resolve `.` and `..` components in paths, making it easier to check for traversal attempts.
        * **Filtering Invalid Characters:** Remove or replace characters that are not allowed in directory names on the target operating system (e.g., `/`, `\`, `*`, `?`, etc.).
    * **Use `pathlib` module:** The `pathlib` module (introduced in Python 3.4) offers a more object-oriented and safer way to handle paths. It can help you construct paths more robustly and handle exceptions.

### 2. Permission Issues

Too often Python programs have too many privileges when executed!

* **Risk:**
    * **Overly Permissive Directories:** If you create directories with overly permissive modes (e.g., `0o777` meaning read, write, and execute for everyone), it can allow other users or processes on the system to read, write, or delete files within that directory. This could lead to:
        * **Data Tampering:** Malicious users could modify or delete important data.
        * **Code Injection:** If your program later executes files from this directory, an attacker could inject malicious code.
    * **Insufficient Permissions:** If your program runs with limited permissions and tries to create a directory in a protected location, it will fail, potentially causing a DoS for your application.

### 3. Race Conditions

* **Risk:** When you check if a directory exists (`os.path.exists()`) and then try to create it (`os.makedirs()`), there is a small window of time (a "race condition") where another process or thread could create the directory, or even a file with the same name, before your `makedirs` call. This can lead to `FileExistsError` or `IsADirectoryError`, or worse, your program interacting with an unintended file.

A simple mitigation is:
* Use `exist_ok=True`: The `os.makedirs()` function (and `pathlib.Path.mkdir()`) has an `exist_ok=True` argument (available in Python 3.2+). This tells the function not to raise an error if the target directory already exists. This is generally the safest way to create directories if you do not need to specifically handle the "already exists" case.
* Robust Error Handling: Even with `exist_ok=True`, it is good practice to wrap directory creation in `try-except` blocks to catch `PermissionError` or other `OSError` exceptions that might occur due to insufficient disk space, invalid path segments, etc.

## Preventive measures

Some simple mitigations that should be present are:

1. Use `os.makedirs(path, mode, exist_ok=True)` or `pathlib.Path(path).mkdir(parents=True, exist_ok=True)`: These are the preferred and safest ways to create directories, handling recursive creation and avoiding errors if the directory already exists.
2. Set appropriate permissions (`mode`) for the application.
3. Use `tempfile.mkdtemp()` for secure temporary directory creation within Python.
4. Monitor storage used by the program and set boundaries.
5. Use the Principle of Least Privilege: **Run your Python program with the minimum necessary permissions on the operating system!**
6. Implement error handling: Use `try-except` blocks to gracefully handle potential `OSError` exceptions (e.g., `PermissionError`, `FileExistsError`, `FileNotFoundError`).


## Example

### Insecure Directory Creation

The following code contains multiple security weaknesses:

```python
import os

# INSECURE: Directly using user input to create a directory
user_input = input("Enter directory name to create: ")

# No input validation, no path sanitisation
os.mkdir(user_input)

# INSECURE: Overly permissive directory (world-writable)
os.makedirs("/tmp/app_cache", mode=0o777)

# INSECURE: Race condition pattern (check-then-act)
if not os.path.exists("/tmp/myapp/data"):
    os.makedirs("/tmp/myapp/data")
```

**What is wrong with this code?**

| Weakness | Explanation |
|----------|-------------|
| No input validation | User could enter `../../../etc/critical` to create directories outside the intended scope |
| No path sanitisation | Special characters or path traversal sequences are not filtered |
| Overly permissive mode (`0o777`) | Any user on the system can read, write, or execute files in this directory |
| Race condition | Between `os.path.exists()` and `os.makedirs()`, another process could create the directory or a malicious file |

**Example attack:** If an attacker supplies `../../../../home/user/.ssh` as input, they might create a directory that interferes with SSH key storage, leading to credential theft or unauthorised access.



### Quick Reference: Insecure vs Secure Patterns in Python

| Insecure Pattern | Secure Replacement |
|------------------|---------------------|
| `os.mkdir(user_input)` | Validate input, use `Path(base) / sanitised`, then `mkdir()` |
| `os.makedirs(path, mode=0o777)` | `path.mkdir(mode=0o700)` or `0o750` |
| `if not os.path.exists(path): os.makedirs(path)` | `path.mkdir(exist_ok=True)` (atomic) |
| Creating directories in `/tmp` directly | `tempfile.mkdtemp()` for temporary directories |
| Hardcoded paths without validation | Resolve paths and verify they remain within a base directory |




## More information

* [Python `os` module documentation](https://docs.python.org/3/library/os.html)

