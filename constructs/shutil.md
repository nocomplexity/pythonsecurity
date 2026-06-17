# Shutil Statement

:::{caution}
Several `shutil` functions can introduce security vulnerabilities, particularly path traversal attacks where files are created or accessed outside of the intended target directory. 

**Always validate file paths before using `shutil` functions with untrusted input. Never pass unsanitised user-controlled paths to these functions.**
:::

## Security Concerns

The `shutil` module provides high-level file operations, but several of its functions can introduce significant security weaknesses when used with untrusted or user-controlled input. The primary concern is **path traversal**, where an attacker can manipulate file paths to access, create, or delete files outside of the intended directory.

The following `shutil` functions require careful security consideration:

**Archive extraction and file operations:**
- `shutil.unpack_archive()` – Vulnerable to path traversal when extracting archives, as members may contain absolute paths or relative paths with `..` sequences that escape the extraction directory.
- `shutil.copy()` – Can overwrite arbitrary files if the destination path is user-controlled.
- `shutil.copy2()` – Same as `copy()`, but also preserves metadata; equally dangerous with unsanitised paths.
- `shutil.copytree()` – Can recursively copy files to unintended locations if source or destination paths are manipulated.

**File system modification:**
- `shutil.chown()` – Can change file ownership of arbitrary files if the path is user-controlled, potentially leading to privilege escalation.
- `shutil.rmtree()` – Can delete arbitrary directories and their contents. **This is extremely dangerous** with unsanitised input, as it could delete critical system files or application data. However this call is/will be depreciated within the shutil module. 

### Path Traversal Attacks

When using functions like `unpack_archive()` with untrusted archives, a malicious archive can contain:
- Absolute paths (e.g., `/etc/passwd`) that reference system files.
- Relative paths with `..` sequences (e.g., `../../sensitive/file`) that escape the extraction directory.

If these paths are not validated, files can be created, overwritten, or deleted outside of the intended location, potentially compromising the entire system.

### Destructive Operations

Functions like `rmtree()` are particularly dangerous because they recursively delete directories without confirmation. If an attacker can influence the path argument, they could:
- Delete application configuration files, causing service disruption.
- Delete user data, leading to permanent data loss.
- Delete system files, causing operating system instability or failure.

## Preventive Measures

To mitigate these risks when using `shutil` functions, implement the following defence-in-depth measures:

1. **Validate all file paths:** Before using any `shutil` function with user-controlled input, sanitise and validate the path:
   ```python
   import os
   
   def is_safe_path(path, base_dir):
       """Check if path is contained within base_dir."""
       abs_path = os.path.abspath(path)
       abs_base = os.path.abspath(base_dir)
       return abs_path.startswith(abs_base + os.sep)
   ```

2. **Use safe extraction functions:** For `unpack_archive()`, validate each member before extraction. Consider using libraries like `defusedxml` or `patool` that provide safer extraction capabilities.

3. **Apply least privilege:** Run your application with minimal filesystem permissions. If using `rmtree()` or `copytree()`, ensure the process only has write access to the directories it needs.

4. **Avoid user-controlled paths:** Where possible, avoid using user input directly as file paths. Use safe, predefined paths with user-provided filenames mapped through a whitelist.

5. **Implement confirmation for destructive operations:** For functions like `rmtree()`, require explicit confirmation from an administrator or authenticated user before proceeding.

6. **Use temporary directories:** For operations like `unpack_archive()`, extract to temporary directories (using `tempfile.TemporaryDirectory`) that are automatically cleaned up and isolated from the rest of the system.

## Example

Consider this vulnerable code that extracts an untrusted archive to a user-specified directory:

```python
"""Vulnerable code - DO NOT USE"""
import shutil

def unpack_vulnerable(archive_path, extract_dir):
    # This is vulnerable to path traversal!
    shutil.unpack_archive(archive_path, extract_dir)
```

A secure implementation validates each file before extraction:

```python
"""Secure archive extraction with path validation"""
import os
import shutil
import zipfile

def safe_unpack_archive(archive_path, extract_dir):
    """Safely extract an archive with path traversal protection."""
    # Sanitise extraction directory
    extract_dir = os.path.abspath(extract_dir)
    os.makedirs(extract_dir, exist_ok=True)
    
    # Validate archive members before extraction
    if archive_path.endswith('.zip'):
        with zipfile.ZipFile(archive_path, 'r') as zf:
            for member in zf.infolist():
                target_path = os.path.join(extract_dir, member.filename)
                abs_target = os.path.abspath(target_path)
                
                # Check that the target is within the extraction directory
                if not abs_target.startswith(extract_dir + os.sep):
                    raise ValueError(f"Unsafe path detected: {member.filename}")
            
            # All paths safe, proceed with extraction
            zf.extractall(extract_dir)
    else:
        # For other archive types, use shutil with caution
        # Consider using a safer library like defusedxml for tar files
        shutil.unpack_archive(archive_path, extract_dir)
```

For `rmtree()` operations, always validate and confirm:

```python
"""Safe directory removal with validation"""
import os
import shutil

def safe_rmtree(directory, allowed_roots):
    """Safely remove a directory tree with path validation."""
    # Ensure directory is absolute and within allowed roots
    abs_dir = os.path.abspath(directory)
    
    if not any(abs_dir.startswith(os.path.abspath(root) + os.sep) 
               or abs_dir == os.path.abspath(root) 
               for root in allowed_roots):
        raise ValueError(f"Directory {directory} is not within allowed locations")
    
    # Optional: confirm before deletion
    confirm = input(f"Are you sure you want to delete {directory}? (yes/no): ")
    if confirm.lower() == 'yes':
        shutil.rmtree(abs_dir)
    else:
        print("Deletion cancelled.")
```

For copy operations, validate both source and destination:

```python
"""Safe copy operation with path validation"""
import os
import shutil

ALLOWED_DIRECTORIES = ['/var/www/uploads', '/tmp/safe']

def safe_copy(src, dest):
    """Safely copy files with path validation."""
    abs_src = os.path.abspath(src)
    abs_dest = os.path.abspath(dest)
    
    # Validate source is within allowed directories
    if not any(abs_src.startswith(os.path.abspath(d) + os.sep) 
               for d in ALLOWED_DIRECTORIES):
        raise ValueError(f"Source {src} is not in allowed locations")
    
    # Validate destination is within allowed directories
    if not any(abs_dest.startswith(os.path.abspath(d) + os.sep) 
               for d in ALLOWED_DIRECTORIES):
        raise ValueError(f"Destination {dest} is not in allowed locations")
    
    shutil.copy2(abs_src, abs_dest)
```

## Discussion

The root cause of security issues in `shutil` functions is that they operate directly on the filesystem without validation of paths or user input. While `shutil` is designed for convenience and flexibility, this flexibility can be dangerous when combined with untrusted data.

The `shutil.unpack_archive()` function is particularly problematic because it automatically determines the archive type and extracts it without path validation. This makes it vulnerable to path traversal attacks, similar to vulnerabilities in the underlying `zipfile`, `tarfile`, and `gzip` modules.

**Note on `shutil.rmtree()`:** This function is considered dangerous in security-sensitive contexts. While it is not deprecated in current Python versions, security audits often flag its usage. For production code, consider:

- Using `pathlib.Path.rmdir()` for empty directories.
- Implementing custom deletion functions with granular permission checks.
- Running deletion operations in a sandboxed environment.

A common misconception is that simply checking the extension or file type of an archive is sufficient to prevent attacks. Attackers can easily rename malicious archives, and path traversal can occur regardless of the archive format. Always validate the actual contents, not just metadata.

For modern Python applications, consider using `pathlib` for safer path manipulation, as it provides better abstraction and some built-in protections against path traversal:

```python
from pathlib import Path

def safe_unpack_with_pathlib(archive_path, extract_dir):
    """Extract archive using pathlib for safer path operations."""
    extract_path = Path(extract_dir).resolve()
    
    # Use pathlib's resolve() to normalize paths
    # Additional validation may still be needed for archive contents
    shutil.unpack_archive(archive_path, extract_path)
```

## More Information

- [Python `shutil` Documentation](https://docs.python.org/3/library/shutil.html)
- [Python `pathlib` Documentation - Safer Path Handling](https://docs.python.org/3/library/pathlib.html)
- [CWE-22: Improper Limitation of a Pathname to a Restricted Directory](https://cwe.mitre.org/data/definitions/22.html)
- [CWE-73: External Control of File Name or Path](https://cwe.mitre.org/data/definitions/73.html)
