---
title: Multiprocessing Module `Connection.recv()` 
short_title: Multiprocessing Module
---


The `multiprocessing` module provides powerful tools for inter-process communication (IPC) in Python. However, its use of the `pickle` module for serialization introduces significant security risks if not handled with care.

## Security Concerns

The `Connection.recv()` method (and its counterparts like `recv_bytes()`) automatically unpickles the received data. Deserializing untrusted data with `pickle` can lead to arbitrary code execution, making it a high-severity vulnerability when the source of the data cannot be fully trusted.

This risk exists because `pickle` is **not** a safe serialization format for untrusted input. Maliciously crafted data can trigger the execution of arbitrary Python code during unpickling.

Even when using `send()` / `recv()` pairs within the same application, design flaws (such as insufficient input validation or exposure of listening endpoints) can allow attackers to inject harmful payloads.

:::{important}
The `multiprocessing` module **always** requires careful design, thorough code review, and robust security controls.
:::


## Preventive Measures

* **Use `Pipe()` where possible**: Connections created with `multiprocessing.Pipe()` are considered safer within a single process tree because both ends are typically controlled by the same application. Even then, treat data as untrusted.

+++

* **Implement strong authentication**: Before calling `recv()` on any `Connection` object (especially those from network listeners, queues, or shared resources), perform proper authentication of the remote process. This may include cryptographic verification, mutual TLS, or other access control mechanisms.

+++

* **Avoid `recv()` on untrusted connections**: Prefer safer alternatives such as:
  * Structured data formats like JSON, MessagePack (with schema validation), or Protocol Buffers.
  * Higher-level libraries that handle authentication and secure serialization.

+++

* **Sandbox or restrict privileges**: Run processes using the `multiprocessing` module with the principle of least privilege.

+++

* **Input validation**: Always validate and sanitise data *after* receiving but *before* any further processing, even if authentication is in place.

+++

* **Prefer `send_bytes()` / `recv_bytes()` with manual deserialization**: This gives you more control, though it does not eliminate the need for careful handling.

## Example

```python
from multiprocessing import Pipe, Process

def worker(conn):
    # Good practice: send trusted data only
    conn.send({"command": "hello", "value": 42})  # Simple dict, safe

def main():
    parent_conn, child_conn = Pipe()
    p = Process(target=worker, args=(child_conn,))
    p.start()

    # Safe because we control both ends and use Pipe()
    data = parent_conn.recv()  # Still verify structure in production!
    print(data)

    p.join()
```

**Warning example (to avoid)**
Dangerous example: receiving from an untrusted or network-exposed connection

```python
conn = ...  # from Listener or external source
data = conn.recv()  # Arbitrary code execution possible!
```

## Discussion

The core issue stems from `pickle`'s design: it can execute code via `__reduce__` methods and other mechanisms. The Python documentation explicitly warns about this risk.

While `Pipe()` reduces the attack surface in simple cases, any exposure to external input, shared memory, or less-controlled process relationships reintroduces the danger. Modern Python security practices favour explicit, auditable serialization over `pickle` for cross-process or cross-machine communication.

Code audit tools relying on Python's `ast` module (without full type inference or data-flow analysis) can only detect common patterns. They may miss sophisticated uses of `multiprocessing.Connection` objects stored in variables or passed through helper functions.

## More Information

* [Python Documentation: multiprocessing](https://docs.python.org/3/library/multiprocessing.html#multiprocessing-recv-pickle-security)