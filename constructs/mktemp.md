# Mktemp Statement

The `tempfile.mktemp()` function in Python's standard library is **deprecated and insecure**. Its use introduces a critical **Time-of-Check to Time-of-Use (TOCTOU)** vulnerability that can be exploited by attackers to compromise your application.

:::{danger} 
**Never use `tempfile.mktemp()` in new code.** The function has been deprecated since Python 2.3 (2003) and removed from Python 3.12+ documentation as a recommended practice. Its continued use represents a significant security risk with no valid justification in modern applications.
:::

## Security Concerns

### Time-of-Check to Time-of-Use (TOCTOU) Vulnerability

The fundamental flaw in `mktemp()` is the gap between generating a filename and actually using it:

```python
# VULNERABLE PATTERN
filename = tempfile.mktemp()  # Step 1: Generate unique-sounding name
# --- TIME GAP --- Attacker can create file here
with open(filename, 'w') as f:  # Step 2: Use the filename
    f.write(data)
```

During this time gap, an attacker can:
1. **Create the file** with malicious content before your application does
2. **Create a symbolic link** pointing to a critical system file (e.g., `/etc/passwd`)
3. **Replace the file** after your application creates it but before it's fully written
4. **Create a directory** with the same name, causing your application to fail or behave unexpectedly

### Race Condition Exploitation

The TOCTOU vulnerability enables several attack vectors:

- **Symlink Attacks**: An attacker creates a symbolic link from the temporary filename to a sensitive file. When your application opens the file for writing, it overwrites the target instead.
- **File Pre-creation**: An attacker creates the file before your application, potentially with malicious content or permissions that your application later reads.
- **Denial of Service**: An attacker creates the file or directory, causing your application to crash or fail when attempting to create it.
- **Data Leakage**: An attacker reads the file contents after creation but before your application secures them.

### No Atomicity Guarantees

`mktemp()` provides **no guarantees** about:
- **Uniqueness**: The generated name is based on predictable patterns
- **Exclusivity**: The file may not exist at creation time but could be created by an attacker
- **Permissions**: Default permissions may be too permissive, exposing sensitive data
- **Path Security**: The function does not verify the security of the parent directory

### Predictable Name Generation

The name generation algorithm in older Python versions was predictable, allowing attackers to:
- Pre-emptively create files with known names
- Calculate future filenames and exploit them
- Create denial-of-service conditions by pre-creating many files

```python
import tempfile

# These patterns might be predictable
file1 = tempfile.mktemp()  # /tmp/tmp123456
file2 = tempfile.mktemp()  # /tmp/tmp123457
# Attacker could guess the pattern and pre-create files
```


## Preventive Measures

### Use `tempfile.mkstemp()` Instead

The safe alternative to `mktemp()` is `tempfile.mkstemp()`, which atomically creates a temporary file with exclusive access:

```python
import tempfile
import os

# SECURE: mkstemp() creates the file atomically
fd, filename = tempfile.mkstemp(suffix='.txt', prefix='myapp_', dir='/secure/temp')

try:
    # Write data using the file descriptor
    with os.fdopen(fd, 'w') as f:
        f.write("Secure data")
    
    # Use the filename if needed
    print(f"Created secure temp file: {filename}")
finally:
    # Clean up
    os.unlink(filename)
```

### Use Context Managers for Automatic Cleanup

For even better security and resource management, use `tempfile.TemporaryFile()` or `tempfile.NamedTemporaryFile()`:

```python
import tempfile

# SECURE: Automatic cleanup and secure creation
with tempfile.TemporaryFile(mode='w+', suffix='.txt', prefix='myapp_') as f:
    f.write("Sensitive data")
    f.seek(0)
    data = f.read()
# File is automatically closed and deleted on context exit

# With a named file (visible in filesystem)
with tempfile.NamedTemporaryFile(mode='w+', delete=True, suffix='.txt') as f:
    f.write("Another secure use case")
    # File remains accessible via f.name, but is deleted on close
```

### Use `tempfile.TemporaryDirectory` for Secure Directories

For temporary directories, use the secure context manager:

```python
import tempfile
import os

# SECURE: Creates and automatically cleans up temporary directory
with tempfile.TemporaryDirectory(prefix='myapp_') as temp_dir:
    temp_file = os.path.join(temp_dir, 'data.txt')
    with open(temp_file, 'w') as f:
        f.write("Secure temporary data")
    # Directory and all contents are removed on exit
```

### Implement Defensive Practices

When creating temporary files, always:

1. **Set Appropriate Permissions**: Use `os.umask()` to set restrictive permissions
2. **Verify Directory Security**: Ensure the target directory is not world-writable
3. **Use Absolute Paths**: Avoid relying on relative paths or environment variables
4. **Clean Up**: Always delete temporary files after use, even in error cases
5. **Limit Exposure**: Minimize the time temporary files exist in the filesystem

### Use Secure Configuration

```python
import tempfile
import os
import stat

def secure_temp_file(data, prefix='myapp_'):
    """
    Create a secure temporary file with restrictive permissions.
    
    Args:
        data: Content to write to the file
        prefix: Prefix for the temporary filename
    
    Returns:
        The filename of the temporary file
    """
    # Set restrictive umask temporarily
    old_umask = os.umask(0o177)
    
    try:
        # Create secure temporary file
        fd, filename = tempfile.mkstemp(prefix=prefix, text=True)
        
        # Set restrictive permissions (owner read/write only)
        os.fchmod(fd, stat.S_IRUSR | stat.S_IWUSR)
        
        # Write data
        with os.fdopen(fd, 'w') as f:
            f.write(data)
        
        return filename
    finally:
        # Restore original umask
        os.umask(old_umask)
```


## Discussion

**Historical Context**

`tempfile.mktemp()` was deprecated in Python 2.3 (2003) after security researchers identified widespread vulnerabilities in applications using the function. Despite being deprecated for over two decades, the function occasionally appears in legacy codebases and beginner tutorials, leading to continued security risks.

## More Information

- [Python `tempfile` Documentation](https://docs.python.org/3/library/tempfile.html)
- [CWE-367: Time-of-check Time-of-use (TOCTOU) Race Condition](https://cwe.mitre.org/data/definitions/367.html)
- [CWE-377: Insecure Temporary File](https://cwe.mitre.org/data/definitions/377.html)
- [CWE-379: Creation of Temporary File in Directory with Incorrect Permissions](https://cwe.mitre.org/data/definitions/379.html)
