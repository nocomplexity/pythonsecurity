# OS System Calls

Direct use of low-level operating system interfaces from the `os` module can be convenient, but it often bypasses Python’s safer, higher-level abstractions and introduces serious security risks.

:::{danger}
Be suspicious of direct `os.*` system calls in Python code. **Never trust — always verify.**
:::

The following Python `os` system calls should by default ring an alarm bell from a security point of view:

* `os.system()` — executes a command in a subshell (equivalent to `subprocess` with `shell=True`).
* The `os.exec*` family (`os.execl`, `os.execle`, `os.execlp`, `os.execlpe`, `os.execv`, `os.execve`, `os.execvp`, `os.execvpe`) — these replace the current process with a new program and **do not return**.
* `os.fork()` — creates a child process (Unix only); can lead to fork bombs or resource exhaustion if misused.
* `os.write()` / `os.writev()` — low-level writes to raw file descriptors.
* And several other similar low-level functions.

:::{tip}
When shell-like functionality is required, prefer the modern `subprocess` module with proper safeguards (see the Subprocess section). Native Python APIs from `os`, `pathlib`, `shutil`, etc., are almost always safer and more portable.
:::

## Security Concerns

The Python `os` functions above are powerful and easy to use, which makes them attractive — but also dangerous. 

**Key risks include:**

- **Command injection** (especially with `os.system()` and similar shell-invoking calls)
- **Process replacement** (the `exec*` family terminates the current Python interpreter)
- **Fork bombs** and resource exhaustion via `os.fork()`
- **File descriptor mismanagement** with `os.write()` / `os.writev()`, which can lead to data corruption, arbitrary file writes, information leakage, or denial of service
- Privilege escalation if these calls run with elevated permissions

These APIs operate at a very low level and provide minimal safety checks compared to higher-level Python constructs.

## Preventive measures

1. **Avoid `os.system()` entirely** — use `subprocess.run()` (or `check_*` variants) with a list of arguments and `shell=False`.

2. **Prefer high-level Python APIs**  
   Use `pathlib`, `shutil`, `os.makedirs()`, `open()`, etc., instead of shelling out for file and directory operations.

3. **Never pass untrusted input** to any `os` function that executes commands or writes to file descriptors.

4. **Validate all inputs rigorously** (paths, filenames, descriptors, etc.). Use allowlists and resolve paths with `pathlib.Path.resolve()`.

5. **Restrict privileges** — run code with the minimum necessary permissions. Avoid running as root.

6. **For `os.write()` / `os.writev()`**:
   - Only use known, validated file descriptors.
   - Prefer Python’s file objects (`open()` / `.write()`) which provide better safety and error handling.

7. **Handle `os.fork()` with extreme care** (if unavoidable) — implement proper resource limiting and error handling to prevent fork bombs.

8. **Consider sandboxing** or running untrusted code in isolated environments when executing system-level operations.

## Example

### Bad example (`os.system`)

**Dangerous — shell injection possible:**

```python
import os


os.system("rm -rf " + user_supplied_path)
```

A simple **Attack scenario** can be:
```python
user_supplied_path = "/tmp/data; rm -rf /"
```

### A bit better example

Example of strict allow list-style validation. Not perfect but a bit more secure.

```python
from pathlib import Path
import subprocess

def safe_remove(path: str) -> None:
    """Safely remove a path after strict validation."""
    target = Path(path).resolve()
    
    
    allowed_root = Path("/safe/directory").resolve()
    if not target.is_relative_to(allowed_root):
        raise ValueError("Path outside allowed directory")
    
    # Use modern subprocess with list arguments
    subprocess.run(["rm", "-rf", str(target)], check=True)
```

Please never ever do a `rm` using a Python system call.You do not needed it, there are far better alternatives!


### More secure example (pure Python, no `rm` or subprocess)

```python
from pathlib import Path
import shutil

def safe_remove(str, allowed_root= "/safe/directory"):
    """Safely remove a file or directory after strict validation.
    
    Uses pure Python standard library functions — no subprocess or shell calls.
    """
    try:
        target = Path(path).resolve(strict=True)     # strict=True raises if path doesn't exist
        root = Path(allowed_root).resolve(strict=True)
        
        # Strict containment check
        if not target.is_relative_to(root):
            raise ValueError(f"Path '{target}' is outside the allowed directory '{root}'")
        
        # Optional: additional explicit allowlist of permitted top-level directories
        # if target.parent != root: ...  # further restrictions if needed
        
        if target.is_file() or target.is_symlink():
            target.unlink(missing_ok=True)
        elif target.is_dir():
            shutil.rmtree(target, ignore_errors=False)   # do not ignore errors #nosec - Can be ignored by SAST scan
        else:
            raise ValueError(f"Path '{target}' is neither a file nor a directory")
            
    except Exception as e:
        raise RuntimeError(f"Failed to safely remove '{path}': {e}") from e
```

This example is better and more secure:

- **No subprocess or shell** — completely avoids `rm`, `os.system`, etc.
- Uses `pathlib.Path` + `shutil.rmtree` — the idiomatic, safe Python way.
- `resolve(strict=True)` ensures the path actually exists and resolves symlinks safely.
- Clear exception handling with context.
- `missing_ok=True` on files prevents unnecessary errors.
- `ignore_errors=False` on `rmtree` ensures failures are not silently ignored.
- Easy to extend with more validation (e.g. file type checks, size limits, etc.).

:::{note} Note

**Never use shell commands or low-level system calls when a safe native Python API exists.**
:::


## Discussion

Low-level `os` calls are sometimes necessary for performance or very specific system interactions, but they should be treated as advanced and potentially **hazardous** features. 

In most applications, you can achieve the same goals more securely and portably using Python’s standard library abstractions. When process management or command execution is truly needed, the `subprocess` module (used correctly) is the recommended approach.

Always assume that any direct system call may be misused and design your code with defence-in-depth principles.

## More information

* [Official `os` module documentation](https://docs.python.org/3/library/os.html)
* [CWE-78: OS Command Injection](https://cwe.mitre.org/data/definitions/78.html)
* [Fork bomb / Rabbit virus attacks](https://www.imperva.com/learn/ddos/fork-bomb/)

