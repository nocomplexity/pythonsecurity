---
title: Ctypes Function
short_title: Ctypes Function
---

## Security Concerns

`ctypes` is a foreign function library for Python. It provides C-compatible data types and allows Python code to call functions in shared libraries and Windows DLLs. It also provides low-level access to memory addresses, pointers and native function interfaces.

Using `ctypes` is not inherently insecure. However, it introduces capabilities that can weaken the security of a Python application if they are used incorrectly or without appropriate safeguards.

It is important to distinguish for `ctypes` the difference between a [**weakness**](appendices/definitions) and a [**vulnerability**](appendices/definitions):

The presence of a `ctypes` import or a native library-loading function does not, by itself, establish the existence of a vulnerability. It identifies functionality that may introduce security weaknesses and therefore deserves further investigation.

Potential security weaknesses associated with `ctypes` include:

- **Untrusted native libraries:** Loading native libraries from untrusted sources or locations can introduce malicious code into the application.
- **Library hijacking:** Using relative paths or uncontrolled library search paths can allow an attacker to substitute a legitimate library with a malicious one.
- **Unsafe memory operations:** Incorrect pointer handling, memory allocation or memory manipulation can result in memory corruption and unexpected application behaviour.
- **Information disclosure:** Direct memory access can expose sensitive information held by the Python process.
- **Native code execution:** Calling functions in untrusted or compromised native libraries can allow arbitrary code execution within the application's security context.
- **Insufficient isolation:** Native code does not automatically respect Python-level restrictions or application-level sandboxing mechanisms.

[Python Code Audit](https://nocomplexity.com/codeaudit/) checks for the use of the following native library-loading functions:

- `ctypes.CDLL`
- `ctypes.PyDLL`
- `ctypes.WinDLL`
- `ctypes.OleDLL`
- `ctypes.windll.LoadLibrary`
- `ctypes.cdll.LoadLibrary`
- `ctypes.oledll.LoadLibrary`

These functions provide mechanisms for loading libraries. Their presence is a security-relevant finding, but does not automatically indicate a  vulnerability. The actual security implications depend on how the functions are used, where the libraries originate and whether an attacker can influence the loading process.

:::{warning}
The `ctypes` module provides low-level access to native libraries, memory addresses and function pointers. It can bypass Python's normal memory safety mechanisms and allow the execution of arbitrary native code.

Incorrect or malicious use can corrupt memory, expose sensitive information, cause application crashes or compromise the running process.
:::

**Important:** Detecting the use of `ctypes` does not mean that a vulnerability exists. It identifies a potentially security-relevant condition that must be assessed in its application context.

## Preventive measures

The following preventive measures help reduce the risk of introducing weaknesses when using `ctypes`:

1. **Only load trusted libraries.** Do not load native libraries from untrusted sources. Avoid loading libraries specified directly by user input, external configuration files or other untrusted sources without strict validation.

2. **Use trusted library locations.** Prefer absolute paths to libraries stored in controlled directories. Avoid relying on the current working directory or directories that untrusted users can modify.

3. **Validate library locations.** Where appropriate, use `ctypes.util.find_library()` to locate libraries. However, finding a library does not establish that it is trustworthy. Validate the resolved path and the library's provenance before loading it.

4. **Apply the principle of least privilege.** Run applications that use `ctypes` with the minimum operating system privileges required. Consider process isolation or sandboxing when loading third-party native libraries.

5. **Verify library integrity and provenance.** Obtain native libraries from trusted sources. Verify their integrity using cryptographic hashes or trusted digital signatures where available. From a secure software supply chain perspective, use reproducible builds and controlled dependency management wherever possible.

6. **Review native function calls.** Carefully inspect the functions exposed by loaded libraries. Define appropriate `argtypes` and `restype` values, validate input data and ensure that pointers and memory buffers are used correctly.

7. **Prefer safer alternatives.** Avoid `ctypes` when an equivalent implementation using standard Python functionality or a well-maintained, trusted Python package is available. Only introduce native code when there is a clear technical requirement.

8. **Include native dependencies in security audits.** Review shared libraries and DLLs as part of the application's software composition and vulnerability management processes. Auditing Python source code alone is not sufficient to establish the security of native dependencies.


## Example

The following example illustrates how `ctypes` can load a native library and call a function.

```python
import ctypes

# Load the C standard library on Linux
libc = ctypes.CDLL("libc.so.6")

# Define the expected function signature
libc.abs.argtypes = [ctypes.c_int]
libc.abs.restype = ctypes.c_int

# Call a native C function
result = libc.abs(-10)

print(result)
```

This example uses a standard system library and calls a well-defined function. There is no obvious security weakness in this particular usage.

However, the same mechanism can be used to load and execute functions from untrusted native libraries.

Consider the following example:

```python
import ctypes
import sys

# Library name supplied by an external source
library_name = sys.argv[1]

# Load the specified native library
lib = ctypes.CDLL(library_name)
```

Here, the library name comes directly from a command-line argument. If an attacker can control this argument, they may be able to influence which native library is loaded.

This represents a potential security weakness. Whether it constitutes an actual vulnerability depends on the application's execution environment, library search behaviour, available attacker privileges and the possibility of supplying a malicious library.


## Discussion

The use of `ctypes` is not inherently insecure. But in all cases it is a weakness that requires mitigation measurements.

`Ctypes` module use is required when Python applications need to interface with existing C libraries, operating system APIs or specialised native functionality.

However, loading a native library is fundamentally different from importing an ordinary Python module. Native libraries execute machine code within the Python process and may access or modify its memory.

For Static Application Security Testing (SAST), it is important to distinguish between identifying potentially dangerous functionality and establishing the existence of an exploitable vulnerability.

Detecting the import of `ctypes` is therefore only a starting point. A more comprehensive static analysis approach should identify:

- Native library-loading functions, including `CDLL`, `PyDLL`, `WinDLL` and `OleDLL`.
- Dynamic library-loading operations, including `LoadLibrary`.
- Native function pointer creation using `CFUNCTYPE`, `WINFUNCTYPE` and `PYFUNCTYPE`.
- Direct memory access using functions such as `string_at`, `wstring_at`, `memmove` and `memset`.
- Pointer manipulation using functions such as `addressof`, `cast` and `from_address`.

A security audit should also consider import aliases and dynamically imported modules. For example, `import ctypes as ct` should be recognised as ctypes usage.

These findings indicate the presence of functionality that deserves security review. They should not automatically be classified as vulnerabilities.

The next step is to assess the context in which the functionality is used. Relevant questions include:

- Is the native library obtained from a trusted source?
- Can an attacker influence the library name or loading path?
- Can an attacker modify the directory containing the library?
- Are native function arguments derived from untrusted input?
- Are pointers and memory buffers handled correctly?
- Does the application run with excessive operating system privileges?
- Are appropriate isolation mechanisms in place?

Static analysis of Python code with a good Python SAST scanner can identify potential weaknesses. 

:::{attention}
Most Python SAST scanners have no default validation to detect the use of `ctypes` library loading.
:::


## More information

- [Python documentation: ctypes — A foreign function library for Python](https://docs.python.org/3/library/ctypes.html)
- [Technical Malware Analysis Report: Python-based RAT Malware](https://www.cyfirma.com/research/technical-malware-analysis-report-python-based-rat-malware/)
- [Analysis of APT37 NarwhalRAT Leveraging MS-Themed Phishing and Dead-drop C2](https://www.genians.co.kr/en/blog/threat_intelligence/narwhalrat)
- [CVE-2026-102734 — National Vulnerability Database](https://nvd.nist.gov/vuln/detail/cve-2026-102734)
- [CVE-2026-37008: CrewAI Python Sandbox Bypass RCE Vulnerability](https://www.sentinelone.com/vulnerability-database/cve-2026-37008/)
- [Python Code Audit — GitHub repository](https://github.com/nocomplexity/codeaudit)


