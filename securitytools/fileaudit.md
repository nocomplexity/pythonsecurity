---
title: Python File Audit
short_title: Python File Audit
---

Handling untrusted input files is one of the most common vectors for security vulnerabilities in Python applications. Accepting file uploads or processing third-party data without explicit validation violates zero-trust principles and exposes your system to Denial of Service (DoS), Remote Code Execution (RCE), and Arbitrary File Write attacks.

The [`fileaudit`](https://github.com/nocomplexity/fileaudit) library provides a lightweight, [open-source](https://nocomplexity.github.io/fileaudit/license.html#software-license) defense layer designed to intercept and validate files before your application processes them.

## File-Based Threat Vectors

When your application handles untrusted input, the attack surface varies depending on the archive or serialisation format.

### Archive Exploits (ZIP, TAR, GZ)

- **Archive / Decompression Bombs:** A heavily compressed file expands exponentially upon extraction (e.g., a few kilobytes expanding into hundreds of gigabytes), quickly exhausting disk space and memory.
- **Inode & DoS Bombs:** Archives containing millions of microscopic files designed to consume all available filesystem inodes or exhaust CPU during unpacking.
- **Path Traversal (Zip Slip):** Archive entries containing relative paths (e.g., `../../etc/passwd`) designed to overwrite system or application files outside the extraction directory.
- **Symlink & Hardlink Attacks:** Archives embedded with symbolic or hard links targeting sensitive host paths, allowing subsequent file writes to redirect to critical system locations.
- **Device Node Injection:** Archives containing character devices, block devices, or FIFOs (named pipes) intended to cause hangs or facilitate privilege escalation.
- **Directory Traversal Depths:** Deeply nested folder structures intended to trigger recursive extraction stack overflows or degrade filesystem operations.

### XML Processing Exploits

Processing untrusted XML documents using standard parsers exposes the application to severe parser-level attacks:

- **Billion Laughs (Entity Expansion):** Exponential inline DTD entity expansion that consumes all available RAM in seconds.
- **XML External Entity (XXE):** Misconfigured DTD parsers disclosing local system files or making SSRF calls via URI handlers.
- **Attribute & Structure Bombs:** Elements containing thousands of attributes or extreme nesting depths designed to trigger stack overflows and CPU starvation.

### JSON & Structured Data Exploits

As highlighted in the [official Python documentation](https://docs.python.org/3/library/json.html), native decoders can be vulnerable to resource exhaustion when parsing malformed or excessively large payloads:

:::{warning}
Unbounded JSON strings or deeply nested objects can force `json.loads()` to consume excessive CPU cycles and RAM. Enforcing strict size and recursion limits prior to parsing is required when dealing with untrusted origins.
:::

## Defending Files with `fileaudit`

The `fileaudit` library mitigates these risks by applying safe extraction constraints, path sanitisation, and size boundaries prior to processing.

### Supported Formats

| Extension | Target File Format | Primary Enforced Protections |
| :--- | :--- | :--- |
| `.csv` | Tabular Data | Length limits, delimiter boundaries |
| `.gz`, `.tgz`, `.tar.gz` | Compressed Archives | Decompression ratio caps, size limits |
| `.json` | JSON Documents | String length, structure depth, raw size |
| `.py` | Python Source Files | File size, static path checks |
| `.tar` | Tape Archives | Member counts, symlink/FIFO rejection |
| `.xml` | XML Documents | Entity expansion bounds, node depth limits |
| `.zip` | ZIP Archives | Path traversal blocking, size limits |

### Key Protections Enforced

:::{admonition} Built-in Security Controls
:class: tip

- **Decompression Caps:** Monitors GZip expansion ratios to catch decompression bombs early.
- **Archive Size & Member Bounds:** Imposes hard upper limits on total extracted size, entry counts, and individual file sizes.
- **Path & Type Sanitisation:** Automatically strips path traversal sequences (`../`), rejecting symlinks, hard links, FIFOs, and special device nodes.
- **Structural Bounds:** Restricts maximum filename lengths, total directory depth, and nesting levels.
:::

## Implementation

### Installation

Install [`fileaudit`](https://github.com/nocomplexity/fileaudit) from [PyPI](https://pypi.org/project/fileaudit/):

```bash
pip install fileaudit

```

### Usage Modes

Each format validator (such as `validate_tar_gz`, `validate_zip`, or `validate_json`) supports two integration patterns depending on your architecture.

#### 1. Decorator Mode

Wrap processing functions directly. The validator inspects the input parameter and aborts execution before the function body runs if checks fail:

```python
from fileaudit import validate_tar_gz

@validate_tar_gz
def process_upload(file_path: str) -> None:
    # Function executes only if the archive passes all security bounds
    ...

```

#### 2. Direct Functional Call

Perform explicit programmatic verification within your validation pipeline:

```python
from fileaudit import validate_tar_gz

def handle_incoming_file(file_path: str) -> None:
    if not validate_tar_gz(file_path):
        raise ValueError("File failed security audit checks.")
        
    # Safe to proceed with extraction
    ...

```

For advanced configuration, parameter tuning, and custom threshold adjustments, consult the official [Python File Audit Documentation](https://nocomplexity.github.io/fileaudit/intro.html).

