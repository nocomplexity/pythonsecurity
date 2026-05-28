---
title: Python bytecode security concerns
short_title: Bytecode and security
---

Python is not secure by default. And based on the [Python Execution Model](executionmodel) it is good to be aware of the most Common Python bytecode security concerns:

## Lack of Bytecode Verification / Trust in Malformed Bytecode
CPython does **not** perform strong validation on bytecode. It largely trusts that the bytecode is well-formed. Malformed or maliciously crafted bytecode (via `.pyc` files or `marshal.load()`) can cause interpreter crashes, memory corruption, or even security exploits (e.g., stack under/overflows or accessing C-level structures).

This is a known design choice: if an attacker can already execute arbitrary bytecode, the Python process is already considered compromised in most threat models.

## Easy Decompilation and Reverse Engineering
Python bytecode (`.pyc` files) is straightforward to decompile back into readable Python-like source code using tools like `uncompyle6`, `pycdc`, `PyLingual`, or others. This makes bytecode-based distribution a very weak form of intellectual property protection or obfuscation.

## Supply Chain Attacks via `.pyc` Files
Attackers can hide malicious code in compiled bytecode (`.pyc`) files uploaded to PyPI or other repositories. Many security scanners only analyze `.py` source files, allowing PYC-only malware to evade detection. Once installed and executed, the bytecode runs directly.

## Insecure Deserialization via `marshal`
The `marshal` module (used internally for `.pyc` files) is **not safe** for untrusted data. Loading maliciously crafted marshaled bytecode/objects can lead to code execution or interpreter instability. (Similar risks exist with `pickle`, but `marshal` is specifically tied to bytecode.)

## Weak Obfuscation / Code Protection
Developers sometimes rely on shipping only `.pyc` files for "protection," but this is easily defeated. Combined with dynamic features like `exec()`, `eval()`, or code object manipulation (`types.CodeType`), it creates additional attack surfaces.

:::{tip}
Never ever run `.pyc` files of others. Just make sure you compile the Python code your self. 
:::

Some companies still practice security-by-obscurity and distribute `.pyc` files, in the hope that their code remains secret. This is a fallacy and a very bad practice!


## Runtime Manipulation of Code Objects
Python allows introspection and modification of code objects at runtime (e.g., via `dis` module or direct bytecode editing). This can be abused for hooking, injection, or bypassing controls in sandboxed/ restricted environments.

