# Subprocess Statement


Using Python's built-in `subprocess` module is extremely powerful for executing external commands and managing child processes. However, it also introduces significant security risks if misused.

:::{caution}
When using the `subprocess` library, your Python code can invoke arbitrary external executables on the host system.
:::


Python's `subprocess` module provides no mechanism to attach to the standard I/O streams of an already-running external process.

## Security Concerns

Legacy subprocess functions carry a **high** severity rating. You should migrate away from the older APIs to the modern `subprocess.run()` (or `subprocess.check_*` variants where appropriate).

**Functions that should be avoided or reviewed carefully**:

* `subprocess.call()`
* `subprocess.Popen()` (when used directly)
* `subprocess.check_call()`
* `subprocess.check_output()`
* `subprocess.getoutput()`
* `subprocess.getstatusoutput()`

## Rationale

The `subprocess` module allows Python code to execute external system commands. This capability is powerful but inherently dangerous, as it creates a direct bridge between application logic (often fed by untrusted input) and the operating system shell.

Common security vulnerabilities arise when:

- User-controlled input is interpolated into command strings
- The shell is invoked (`shell=True`)
- Arguments are passed as strings instead of lists
- Output or exit codes are trusted without proper validation
- Errors are silently ignored or mishandled

If an attacker can influence any part of the constructed command, they may achieve **command injection**, privilege escalation, data exfiltration, or remote code execution (RCE).

All listed APIs are safe **by default only when used correctly** — but they are very easy to misuse.

## Preventive measures

These guidelines apply to **all** `subprocess` APIs:

1. **Avoid `shell=True` whenever possible**  
   This is the single biggest risk factor. Shell metacharacters (`;`, `&&`, `|`, `$()`, `` ` ``, etc.) become exploitable when the shell is involved.

2. **Always pass arguments as lists, never as strings**  
   ```python
   # Good
   subprocess.run(["ls", "-l", "/tmp"])
   
   # Bad
   subprocess.run("ls -l /tmp", shell=True)
   ```

3. **Never concatenate or interpolate user input into command strings**  
   This applies especially to paths, filenames, flags, or filters.

4. **Validate and sanitise all inputs**  
   Prefer allowlists over blocklists. Consider using libraries like `shlex.quote()` when string construction is unavoidable.

5. **Explicitly check return codes and handle errors**  
   Silent failures can mask partial or malicious command execution.

6. **Run subprocesses with the least privileges possible**  
   Never execute subprocesses as root (or other high-privilege users) unless absolutely necessary.

7. **Prefer high-level Python APIs**  
   Many common shell operations have safe, pure-Python equivalents in `os`, `pathlib`, `shutil`, etc.

## Example

### Bad example (`subprocess.call`)

Highly dangerous example

```python
import subprocess
subprocess.call("rm -rf " + user_input, shell=True)
```

**Attack scenario**:
```python
user_input = "/tmp/data; rm -rf /"
```

This would delete the entire filesystem (if permissions allow).

### A bit better example

```python
import subprocess
from pathlib import Path

def safe_remove(path: str) -> None:
    """Safely remove a file or directory after validation."""
    target = Path(path).resolve()
    
    # Strict validation (example)
    if not target.is_relative_to("/safe/directory"):
        raise ValueError("Path outside allowed directory")
    
    subprocess.run(["rm", "-rf", str(target)], check=True)
```


:::{warning}
**Important Note**

Even this safer-looking code:

```python
subprocess.run(["rm", "-rf", str(target)], check=True)
```

can still introduce security weaknesses if not implemented correctly. This is why a good Python SAST tool, such as [Python Code Audit](https://github.com/nocomplexity/codeaudit), will flag the use of `subprocess.run` (and similar functions). 

An expert security review is still essential, as no automated tool — including AI — can fully understand the specific context in which the code will run.
:::

## Discussion

Executing shell commands or calling external applications from Python is a powerful capability, but it is **inherently dangerous**. 

Wherever possible, you should use a native Python API instead of invoking shell commands. In the vast majority of cases, this is not only feasible but also significantly more secure.

`Popen` is the low-level foundation and used for some advanced cases, but use `run()` for most cases if you need this kind of functionality.

Using `Popen` itself is not directly a [vulnerability](vulnerability), but too often crucial checks are missing so within code `Popen` is often a weakness that should be reviewed in depth.

## More information

* [Official subprocess documentation](https://docs.python.org/3/library/subprocess.html)



