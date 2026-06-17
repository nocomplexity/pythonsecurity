# Random Statement

:::{warning}
The `random` module is **not** suitable for security or cryptographic purposes. Its pseudo-random number generator is deterministic and predictable if an attacker observes enough output. Never use it for generating session tokens, encryption keys, passwords, or any other security-sensitive values.
:::

## Security Concerns

The `random` module in Python is unsafe for security or cryptographic purposes, such as generating session tokens, encryption keys, or passwords.

This is because the `random` module uses a pseudo-random number generator (PRNG) called the Mersenne Twister. This algorithm is **deterministic**—if an attacker can observe a sufficient amount of its output, they can completely determine its internal state (the seed) and accurately predict all future and even past values. This predictability can lead to severe vulnerabilities, including session hijacking, credential compromise, and unauthorised access.

The following `random` functions can introduce weaknesses when used in security contexts:
- `random.seed`
- `random.Random`
- `random.randbytes`
- `random.randint`
- `random.random`
- `random.randrange`
- `random.triangular`
- `random.uniform`

The `random` module is specifically designed for non-security-sensitive applications like simulations, statistical modelling, and simple games, prioritising speed and good statistical distribution over true unpredictability.

For all security-sensitive tasks, you **must** use the `secrets` module, which relies on a Cryptographically Secure Pseudo-Random Number Generator (CSPRNG) provided by the operating system.

## Preventive Measures

To avoid introducing predictable randomness vulnerabilities in your applications, follow these guidelines:

- **Use `secrets` for security purposes:** For any security-sensitive randomness—such as passwords, tokens, or cryptographic keys—always use the `secrets` module. This module provides functions like `secrets.token_bytes()`, `secrets.token_hex()`, and `secrets.choice()` that are designed for cryptographic security.

+++


- **Use `random.SystemRandom` with caution:** The `SystemRandom` class uses `os.urandom()` and can be used as a drop-in replacement for the `random` module's functions. However, it may not be available on all systems, and using `secrets` directly is the preferred, more explicit approach.

+++

- **Review third-party dependencies:** Ensure that any third-party libraries you use are not relying on the `random` module for security-critical operations. Vulnerabilities can be introduced through transitive dependencies.


## Example

Consider the following problematic code that uses `random` to generate a browser cookie:

```python
"""Problematic code using random module"""
import random

browser_cookie = random.randint(min_value, max_value)
```


This code is vulnerable because an attacker could predict the cookie value if they observe enough outputs from the same PRNG instance.

To improve this code, use the `secrets` module:


Use the  `SystemRandom` class. This class uses the system function `os.urandom()` to generate random numbers from sources provided by the operating system.

```python
from random import SystemRandom
safe_random = SystemRandom()

browser_cookie = safe_random.randint(min_value, max_value)
```

Alternatively, if you need a drop-in replacement for `random` functions, use `SystemRandom`:

```python
from random import SystemRandom
safe_random = SystemRandom()

browser_cookie = safe_random.randint(min_value, max_value)
```

However, note that `secrets` is the recommended approach for all new code requiring cryptographic randomness.


## Discussion

The root cause of the security issue lies in the design goals of the Mersenne Twister algorithm. It was developed for speed and statistical quality in simulations, not for unpredictability. Its state can be fully recovered from just 624 consecutive outputs, making it trivial for an attacker to predict future values once they have observed enough random numbers.

The Python standard library provides the `secrets` module specifically to address this gap. Introduced in Python 3.6 via [PEP 506](https://peps.python.org/pep-0506/), `secrets` was designed to be the "go-to" module for all cryptographic randomness needs. It abstracts away platform-specific CSPRNG implementations and provides a simple, safe API for developers.

A common misconception is that using `random.seed()` with a high-entropy seed makes the `random` module secure. This is false—while it may make prediction more difficult initially, the Mersenne Twister's state can still be recovered from its outputs, regardless of the seed's quality.

## More Information

* https://docs.python.org/3/library/random.html
* https://docs.python.org/3/library/secrets.html#module-secrets
* [ CWE-330: Use of Insufficiently Random Values](https://cwe.mitre.org/data/definitions/330.html)
* [CWE-338: Use of Cryptographically Weak Pseudo-Random Number Generator (PRNG)](https://cwe.mitre.org/data/definitions/338.html)
* [CVE-2022-23472](https://nvd.nist.gov/vuln/detail/CVE-2022-23472)
* [PEP 506 – Adding A Secrets Module To The Standard Library](https://peps.python.org/pep-0506/)
* https://www.codiga.io/blog/python-avoid-random/

