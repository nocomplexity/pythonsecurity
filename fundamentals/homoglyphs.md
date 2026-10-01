---
title: Homoglyphs in Python Code
short_title: Homoglyphs
---

:::{note} What are homoglyphs?
Homoglyphs are Unicode characters that look visually identical (or nearly identical) to ASCII characters or to each other.  
Python’s Unicode support allows many of these characters in identifiers, which creates real security risks (see [PEP 672](https://peps.python.org/pep-0672/) and [Unicode Technical Report #39](https://www.unicode.org/reports/tr39/)).

A homoglyph is one of two or more graphemes, characters, or glyphs whose shapes appear identical or very similar ([Wikipedia](https://en.wikipedia.org/wiki/Homoglyph)).
:::

## How Homoglyphs Can Be Misused in Python Code

Homoglyphs introduce security risks in several scenarios. They are a proven technique for hiding malicious code that is difficult for both humans and many automated tools to detect.

### Typosquatting / supply-chain attacks

A malicious package named `reqυests` (using a Greek upsilon `υ` instead of the Latin `u`) can be published to PyPI. Victims who mistype the legitimate package name `requests` may install and import the malicious version.

### Backdooring code

Two functions that look identical can coexist because Python treats them as different names:

```python
def transfer(amount):          # ASCII 'a'
    return real_transfer(amount)

def trаnsfer(amount):          # Cyrillic 'а' — different identifier!
    exfiltrate(amount)         # backdoor
```

### Bypassing security checks

Regex filters and string comparisons that assume ASCII can be trivially bypassed:

```python
>>> "раyment" == "payment"
False
>>> "раyment".isascii()
False
```

### Misuse of built-ins such as `exec`

The following call works because Python applies **NFKC normalisation** to identifiers:

```python
ℯ𝓍ℯ𝒸("print(2 + 2)")   # becomes exec after NFKC normalisation
```

Python normalises identifiers with [NFKC](https://docs.python.org/3/library/unicodedata.html) at parse time, so `ℯ𝓍ℯ𝒸` is treated as `exec`. However, the characters in the source are *not* equal to the literal string `"exec"`, which can fool many static-analysis (SAST) tools.

:::{seealso} What is NFKC normalisation?
:class: dropdown
NFKC stands for *Normalization Form Compatibility Composition*.  
Characters are first decomposed by compatibility equivalence, then recomposed by canonical equivalence.  
See [Unicode equivalence](https://en.wikipedia.org/wiki/Unicode_equivalence) and the authoritative reference [Unicode Normalization Forms (UAX #15)](https://unicode.org/reports/tr15/).
:::

Consequently, the following also executes successfully and is frequently missed by SAST scanners:

```python
ℯ𝓍ℯ𝒸("import os; os.system('rm -rf ~/.*')")
```

## Defensive Tips

To protect your codebase from homoglyph attacks and unintended identifier spoofing, implement the following defensive practices:

* **Enforce ASCII-Only Identifiers:** PEP 8 recommends restricting identifiers to ASCII characters. You can enforce this automatically using modern linters (such as Ruff, Flake8, or Pylint) or with a simple pre-commit hook or CI check that rejects non-ASCII characters in source files.

* **Validate Dynamic Identifiers:** If your application accepts or constructs identifiers dynamically (such as through user input, plugins, or reflection), validate them immediately using `str.isascii()` before execution or lookup.

* **Detect Normalization Anomalies:** Use Python's `unicodedata` module to check if an identifier changes under NFKC normalization or deviates from standard ASCII:

```python
import unicodedata

def is_safe_identifier(identifier: str) -> bool:
    if identifier.isascii():
        return True

    # Flag identifiers that change under NFKC normalization
    normalized = unicodedata.normalize("NFKC", identifier)
    if normalized != identifier:
        return False

    return True
```


* **Configure Review Tooling and SAST:** Ensure that your code-review interfaces, IDEs, and static analysis (SAST) scanners are configured to explicitly highlight non-ASCII characters, zero-width characters, and mixed-script identifiers so reviewers can catch obfuscated code.

:::{note}
Remember that disabling UTF-8 mode (`python -X utf8=0` / `PYTHONUTF8=0`) only affects runtime I/O encoding.

Normally, when UTF-8 Mode is active (enabled via `-X utf8` or automatically in most environments like POSIX `C` locales), and becoming the default in modern Python versions, Python ignores the system locale and forces UTF-8 for things like `open()`, standard input/output streams, and filenames.
:::

:::{danger}
Disabling `UTF8` is **NOT** recommended and a bad security practice! 
It does **not** restrict Unicode in source code and is not a useful defence.
:::
