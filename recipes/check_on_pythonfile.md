---
title: How to safely validate untrusted Python files before parsing
short_title: How to validate Python before parsing
---

## The Challenge

Any service that processes uploaded or third-party `.py` files—SAST tools, CI pipelines, IDEs, code-analysis services, linters, web-based code review platforms, or AI code assistants—must validate untrusted Python source **before** calling `ast.parse()`.

:::{danger}
You cannot safely parse what you have not validated.
:::

Validating a Python file before processing it is a genuine challenge. It requires both security awareness and careful engineering, especially because developers routinely compose applications from modules written by others via a simple `import`.

## The Threat

Python source files can be weaponised *before* any analysis or execution takes place. Path traversal, symlinks, oversized inputs, null bytes, non-UTF-8 encodings, and pathological constructs can exhaust memory, consume unbounded CPU, crash the interpreter, or escape intended directory boundaries. Performing these checks **before** the file is fully read into memory or passed to `ast.parse()` mitigates denial-of-service and injection risks that would otherwise propagate to downstream linters or execution pipelines. Even many SAST tools break on maliciously constructed Python files.

**The chain of trust breaks at byte zero.** A submitted file may be:

- Not Python at all (a binary payload, polyglot, or disguised executable)
- Python, but engineered to exploit the parser or tokenizer itself
- Python, but engineered to exhaust resources when parsed or analysed
- Python, but designed to trigger dangerous side effects the instant anyone *imports* it during your pipeline

Specific hazards include:

- Deeply nested expressions (`((((((…1…))))))`), extremely long operator chains, pathological f-strings, or certain literal constructions can keep the CPU busy for seconds or longer—while still fitting within ordinary size and line limits.
- A single malicious file can pin a worker thread or process, leading to thread-pool starvation, cascading latency, health-check failures, and container restarts.
- Parallel uploads of such files amplify the attack at almost no bandwidth or disk cost to the attacker (a classic asymmetric DoS).
- Post-parse checks (`max_ast_nodes`, handlers for `MemoryError` / `RecursionError`) never run if parsing never completes. The C parser has its own recursion limits; in some edge cases or versions it may raise a clean exception, in others it may exhaust the stack.

**`ast.parse()` is not a free entry point**

- `ast.parse()` is a blocking call into CPython’s C tokenizer and parser over potentially attacker-controlled bytes.
- Python’s parser has historically contained bugs that allowed hangs or crashes on malformed input (for example, exponential backtracking on certain stacked tokens fixed in bpo-46707 / related issues).
- The parser makes implicit assumptions that its input is well-formed and benign—nothing in the format itself enforces this.
- Once `ast.parse()` starts, pure Python code cannot interrupt it. A crafted input can therefore occupy the parser for a long time or crash the process.

CPython currently rejects nesting deeper than approximately 200 levels with a clean `SyntaxError: too many nested parentheses`. Extremely long lines or other pathological constructs can still cause prolonged CPU usage or memory exhaustion. Bounds and a timeout remain essential.

## Vulnerable Code Examples

```python
import ast
import tempfile
import time
from pathlib import Path


def validate_and_parse(path):
    """The naive approach: read and parse without ANY validation.

    VULNERABLE: no size limit, no timeout, no path checks.
    A crafted file can hang this call for a long time or crash the interpreter.
    """
    source = Path(path).read_text(encoding="utf-8")
    # No timeout -- a crafted file can occupy the parser for a long time
    return ast.parse(source, filename=str(path))
```


### oversized / malformed file (may raise on decode or parse)

```pyton
bad_py = Path(tempfile.gettempdir()) / "bad_demo.py"
bad_py.write_bytes(b"\x00\x00 not python at all \xff\xfe" * 100)

try:
    validate_and_parse(bad_py)
    print("Bad file: parsed without complaint (!)")
except (UnicodeDecodeError, SyntaxError, ValueError) as e:
    print(f"Bad file: raised after full read -> {type(e).__name__}: {e}")

```

### Pathological constructs(Malicious inputs)

```python
# Run at your own risk. Restart the process if it becomes unresponsive.
evil_py = Path(tempfile.gettempdir()) / "evil_demo.py"
evil_py.write_text("x = " + "(" * 10_000_000 + "1" + ")" * 10_000_000)
validate_and_parse(evil_py)   # <-- can occupy the parser for a long time
```

Any of the pathological constructs described above can cause prolonged CPU usage, high memory consumption, or (in edge cases) a process crash.

## Secure Mitigation

Use the battle-tested [`fileaudit`](https://github.com/nocomplexity/fileaudit) library. Its `@validate_python` decorator (or the equivalent `validate_python()` function) performs all necessary pre-parse checks **and** applies a bounded `parse_timeout` before the AST is constructed.

```python
import ast
import tempfile
import time
from pathlib import Path

from fileaudit.python_check import validate_python

# ------------------------------------------------------------------
# Configure the guard once -- every limit is configurable,
# but the defaults are already safe for most production services.
# ------------------------------------------------------------------
GUARD = dict(
    max_file_size=10 * 1024 * 1024,   # 10 MB
    max_lines=100_000,
    max_line_length=10_000,
    max_ast_nodes=500_000,
    parse_timeout=10,                 # seconds -- the critical bound (Unix SIGALRM)
    allowed_base_dir=tempfile.gettempdir(),  # path-traversal guard
)


def process(source_path):
    """Only reached once the file has already been validated.

    All the dangerous operations happen behind the decorator:
    bounded size, bounded lines, and a hard timeout on parsing.
    """
    source = Path(source_path).read_text(encoding="utf-8")
    return ast.parse(source, filename=str(source_path))


guarded_process = validate_python(**GUARD)(process)
```

### A perfectly normal file — parses as before

```python
normal_py = Path(tempfile.gettempdir()) / "normal_demo.py"
normal_py.write_text("x = 1 + 2\nprint(x)\n", encoding="utf-8")

t0 = time.time()
tree = guarded_process(normal_py)
print(f"Normal file: parsed in {time.time() - t0:.4f}s "
      f"({len(tree.body)} top-level statements)")
```

Outcome:

```text
Normal file: parsed in 0.0005s (2 top-level statements)
```

### Oversized / malformed file — rejected BEFORE parsing

```python
bad_py = Path(tempfile.gettempdir()) / "bad_demo.py"
bad_py.write_bytes(b"\x00\x00 not python at all \xff\xfe" * 100)

try:
    guarded_process(bad_py)
    print("Bad file: accepted (!)")
except Exception as e:
    print(f"Bad file: rejected before parse -> {type(e).__name__}: {e}")
```

Outcome:

```text
Bad file: rejected before parse -> PythonValidationError: FileAudit Security Validation Failed - File is not valid UTF-8: 'utf-8' codec can't decode byte 0xff in position 21: invalid start byte
```

### Pathological constructs (bounded by limits and timeout)

Extremely deep nesting or very long lines are rejected by the size, line-length, or timeout guards. The example below triggers the line-length limit first:

```python
# Safe to run: the pre-parse checks keep the process responsive
evil_py = Path(tempfile.gettempdir()) / "evil_demo.py"
evil_py.write_text("x = " + "(" * 2_000_000 + "1" + ")" * 2_000_000)

t0 = time.time()
try:
    guarded_process(evil_py)
    print("Evil file: accepted (!)")
except Exception as e:
    print(f"Evil file: rejected in {time.time() - t0:.2f}s "
          f"-> {type(e).__name__}: {e}")

print("Process still responsive -- the bounded-latency invariant holds.")
```

Result:

```text
Evil file: rejected in 0.01s -> PythonValidationError: FileAudit Security Validation Failed - Line 1 length (4000005) exceeds maximum limit of 10000 characters
Process still responsive -- the bounded-latency invariant holds.
```

All limits are configurable, but **the defaults already provide conservative protection** suitable for most production services. Note that `parse_timeout` relies on Unix `SIGALRM` and is therefore most effective on Unix-like systems.

## Discussion

[`fileaudit`](https://github.com/nocomplexity/fileaudit/tree/main) implements a needed timeout—along escape. But this library for validation Python files also validates Python files on size, line-length, encoding, path-containment, and has AST-node guards—behind a simple decorator or function call. 

Adopting it restores the bounded-latency invariant and closes the DDoS vector without requiring every application to re-implement the same low-level signal handling itself.

Using [fileaudit](https://nocomplexity.github.io/fileaudit/python_validations.html) is a simple, secure solution when your Python program processes or analyses Python files.
