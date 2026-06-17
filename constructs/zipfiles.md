# Zipfile Extraction

:::{caution}
Never extract archives from untrusted sources without prior inspection! Files can be created outside of the specified extraction directory—for example, members with absolute filenames starting with "/" or filenames containing ".." can escape the intended target location.

**Always validate archive contents before extraction and implement resource limits to prevent denial of service.**
:::

## Security Concerns

When using Python's `zipfile` module—or any archive extraction functionality—there are significant risks when processing maliciously prepared compressed files. These vulnerabilities can lead to storage exhaustion, path traversal attacks, and denial of service.

The following methods and functions are particularly susceptible:

**Archive extraction functions:**
- `zipfile.extractall()`
- `zipfile.open()`
- `shutil.unpack_archive()`

**Compression libraries:**
- `gzip.open()`
- `bz2.open()` and `bz2.BZ2File()`
- `lzma.open()` and `lzma.LZMAFile()`
- `zstd.decompress()` and `zstd.open()`

### Denial of Service via Resource Exhaustion

If a compressed file is controlled by a malicious user, they can create a highly compressed payload that expands to an enormous size when decompressed—commonly known as a **"zip bomb"** or decompression bomb. Such files can quickly consume all available system memory, causing the application to crash or the server to become unresponsive. This is a particularly common attack vector when processing user-uploaded or external compressed files.

### Path Traversal Vulnerabilities

A path traversal vulnerability can arise when the file path within an archive is constructed from, or influenced by, user input. For example, a malicious archive might contain a member with the path `../../../../etc/passwd`. When extracted, this file could overwrite or access sensitive system files outside of the intended extraction directory. This is a critical security consideration for any code that handles file paths based on external data.

### Data Amplification Attacks

Compression algorithms can produce extreme amplification ratios (up to thousands-to-one). A tiny compressed file of a few kilobytes could expand to gigabytes or even terabytes of data, overwhelming storage capacity and causing system failures.

## Preventive Measures

To mitigate these risks when working with compressed files, implement the following defence-in-depth measures:

1. **Only decompress files from trusted sources:** The most effective measure is to ensure that archive extraction functions are never called on untrusted data. If you must accept user-uploaded archives, treat them as untrusted and apply additional safeguards.

2. **Implement size limits for decompression:** While not always straightforward, set a maximum decompressed size limit. For libraries like `lzma` that lack built-in size limiting, read data in fixed-size chunks and track the total decompressed size, raising an error if it exceeds a predefined limit.

   ```python
   CHUNK_SIZE = 8192
   MAX_SIZE = 100 * 1024 * 1024  # 100 MB limit
   
   total_decompressed = 0
   with lzma.open(malicious_file) as f:
       while chunk := f.read(CHUNK_SIZE):
           total_decompressed += len(chunk)
           if total_decompressed > MAX_SIZE:
               raise ValueError("Decompressed size exceeds limit")
   ```

3. **Check file metadata:** Where possible, read the uncompressed size from the archive header before beginning decompression. While not all formats include this information, it can be a useful first check. **Note: This measure should never be used as the sole safeguard—it is easily bypassed with crafted archives.**

4. **Validate archive members:** Before extraction, inspect all member filenames to prevent path traversal:
   ```python
   import os
   import zipfile
   
   def safe_extract(zip_path, extract_dir):
       with zipfile.ZipFile(zip_path, 'r') as zf:
           for member in zf.infolist():
               # Resolve the absolute path
               target_path = os.path.join(extract_dir, member.filename)
               abs_target = os.path.abspath(target_path)
               abs_extract = os.path.abspath(extract_dir)
               
               # Check that the target is within the extraction directory
               if not abs_target.startswith(abs_extract + os.sep):
                   raise ValueError(f"Attempted path traversal: {member.filename}")
           
           # All members are safe, proceed with extraction
           zf.extractall(extract_dir)
   ```

5. **Resource monitoring:** Monitor application memory, CPU, and resource usage during decompression. Terminate the process if resource consumption exceeds acceptable thresholds. **Note: This measure is not fail-safe and should be used as a secondary defence.**

6. **Use sandboxing:** Consider running archive extraction in a sandboxed environment (such as a Docker container) with limited resources, memory caps, and filesystem restrictions.

## Example

Consider this vulnerable code that extracts an untrusted archive:

```python
"""Vulnerable code - DO NOT USE"""
import zipfile

def extract_vulnerable(zip_path, extract_dir):
    # This is vulnerable to zip bombs and path traversal
    with zipfile.ZipFile(zip_path, 'r') as zf:
        zf.extractall(extract_dir)  # Dangerous!
```

The improved, a more secure version implements multiple safeguards:

```python
"""Secure archive extraction with defence-in-depth"""
import os
import zipfile

MAX_TOTAL_SIZE = 100 * 1024 * 1024  # 100 MB
MAX_FILE_COUNT = 1000

def safe_extract(zip_path, extract_dir):
    with zipfile.ZipFile(zip_path, 'r') as zf:
        total_size = 0
        file_count = 0
        
        for member in zf.infolist():
            # Prevent zip bombs by tracking total size
            total_size += member.file_size
            if total_size > MAX_TOTAL_SIZE:
                raise ValueError(f"Archive exceeds maximum size of {MAX_TOTAL_SIZE} bytes")
            
            # Prevent denial of service via excessive files
            file_count += 1
            if file_count > MAX_FILE_COUNT:
                raise ValueError(f"Archive contains too many files (max {MAX_FILE_COUNT})")
            
            # Prevent path traversal
            target_path = os.path.join(extract_dir, member.filename)
            abs_target = os.path.abspath(target_path)
            abs_extract = os.path.abspath(extract_dir)
            
            if not abs_target.startswith(abs_extract + os.sep):
                raise ValueError(f"Attempted path traversal: {member.filename}")
        
        # All checks passed, safe to extract
        zf.extractall(extract_dir)
```

For gzip files specifically, implement chunk-based decompression with size limits:

```python
"""Secure gzip decompression with size limits"""
import gzip

CHUNK_SIZE = 8192
MAX_SIZE = 50 * 1024 * 1024  # 50 MB limit

def safe_gzip_decompress(data):
    decompressed = b''
    total_size = 0
    
    with gzip.open(data, 'rb') as f:
        while True:
            chunk = f.read(CHUNK_SIZE)
            if not chunk:
                break
            total_size += len(chunk)
            if total_size > MAX_SIZE:
                raise ValueError(f"Decompressed data exceeds {MAX_SIZE} bytes")
            decompressed += chunk
    
    return decompressed
```

## Discussion

The security issues with archive extraction arise from a fundamental tension: compression algorithms are designed to efficiently reduce file sizes, but this efficiency can be weaponised through data amplification attacks. The amplification factor can be enormous—a 42 KB zip bomb (42.zip) famously expands to 4.5 petabytes of data.

The Python standard library's archive modules generally **do not** include built-in protection against these attacks. Each module has different default behaviours and limitations, making it essential for developers to implement their own safeguards.

The `zipfile` module has a documented `BadZipFile` exception, but this does not protect against zip bombs or path traversal—it only handles malformed ZIP file structures. Similarly, `gzip.open()` provides no built-in size limiting or path validation.

For production applications that process user-uploaded archives, consider these additional best practices:

- **Use dedicated security libraries:** Libraries like `defusedxml` have equivalents for archive extraction, providing safe wrappers around standard library functions.
- **Extract to temporary directories:** Use Python's `tempfile` module to create temporary extraction directories that are automatically cleaned up, limiting the impact of resource exhaustion.
- **Implement queue-based processing:** Process archives in a separate worker process with strict memory limits, terminating the process if it exceeds those limits.

A common misconception is that checking file sizes individually is sufficient. A zip bomb can consist of many small files that collectively expand to enormous sizes, or a single highly compressed file with extreme compression ratios. Always validate total decompressed size and implement multiple layers of defence.

## More Information

- [Python `zipfile` Security Considerations](https://docs.python.org/3/library/zipfile.html#zipfile-resources-limitations)
- [Python `gzip` Documentation](https://docs.python.org/3/library/gzip.html)
- [Python `bz2` Documentation](https://docs.python.org/3/library/bz2.html#bz2.open)
- [PEP 784 – zstd Support](https://peps.python.org/pep-0784/)
- [CWE-409: Improper Handling of Highly Compressed Data (Data Amplification)](https://cwe.mitre.org/data/definitions/409.html)
- [CVE-2025-66471 – urllib3 Streaming API Improperly Handles Highly Compressed Data](https://www.cve.org/CVERecord?id=CVE-2025-66471)
- [Zip Bomb (42.zip) – Wikipedia](https://en.wikipedia.org/wiki/Zip_bomb)

