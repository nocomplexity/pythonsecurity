# How To Prevent the Misuse of `assert` in Production Code


## The Challenge

Python’s `assert` statement is often misunderstood and misused as a runtime validation or security mechanism. While `assert` is perfectly valid for debugging and testing, relying on it for **production checks**, **input validation**, or **security controls** introduces hidden risks that can fundamentally change how an application behaves once deployed.

The core problem: **assertions are not guaranteed to execute in production**.

---

## The Threat

From a security standpoint, `assert` is unreliable by design:

- **Assertions can be completely disabled**  
  When Python runs in optimized mode (`python -O` or `python -OO`, or via `PYTHONOPTIMIZE`), *all* `assert` statements are stripped from the bytecode.  
  Any security check implemented with `assert` simply vanishes.

- **Security controls silently disappear**  
  If `assert` is used to validate user input, enforce type checks, prevent invalid states, or block malicious data, attackers can bypass those checks simply because they are not present in optimized environments.

- **Side effects are skipped**  
  Any logic embedded in an `assert` expression—such as function calls or state changes—will not execute at all when assertions are disabled, potentially leaving the application in an unsafe or inconsistent state.

- **Different runtime behavior = exploitable gaps**  
  Code paths may behave safely in development but dangerously in production. This discrepancy is fertile ground for vulnerabilities such as:
  - SQL injection
  - Logic bypass
  - Unexpected crashes (DoS)
  - Data corruption

In short: **`assert` is not a security boundary**.

---

## Vulnerable Code Example

The following example demonstrates how relying on `assert` for runtime validation leads to different—and unsafe—behavior when Python is run with optimizations enabled.

```python
"""
Vulnerable example: using assert for runtime validation.
"""

def divide_numbers(x, y):
    # Intended to prevent division by zero
    assert y != 0, "Invalid divisor"
    return x / y


print("--- Demonstrating danger of assertions ---")

# Works as expected in normal mode
print(divide_numbers(10, 2))

try:
    # When run with `python -O`, the assert line is removed
    # This will raise ZeroDivisionError instead of AssertionError
    print(divide_numbers(10, 0))
except AssertionError as e:
    print(f"Caught AssertionError: {e}")
```

Run the script twice:
```
python assert_example.py
python -O assert_example.py
```

You will observe different execution paths, proving that the safety check is unreliable.

## Secure Mitigation

The only safe alternative in production code is explicit validation with proper exceptions.

```python
"""
Secure example: explicit checks with proper exception handling.
"""

def divide_numbers(x, y):
    if not isinstance(y, (int, float)):
        raise TypeError("Divisor must be a number")

    if y == 0:
        raise ValueError("Divisor must not be zero")

    return x / y


print("--- Secure behavior ---")

try:
    print(divide_numbers(10, 0))
except (TypeError, ValueError) as e:
    print(f"Handled error safely: {e}")
```

- This code behaves consistently
- Works in optimized and non-optimized modes
- Can be logged, handled, or safely propagated
- Suitable for production and security-critical logic

## Discussion

When assert is appropriate:

- Unit tests (pytest, unittest)

- Debug-only sanity checks

- Internal invariants during development



When assert is dangerous:

- User input validation

- Authorization or authentication logic

- Data integrity checks

- Business rules

- Any code that must always run



Key takeaways:

- assert is a debugging aid, not a runtime safeguard.

- Optimized Python removes assertions entirely.

- Security checks that disappear are worse than no checks at all.

- **Explicit condition checks + explicit exceptions** are the only safe pattern.

If a condition matters for correctness, stability, or security, never rely on `assert`.


For more information check:
* [The assert statement - Python Documentation](https://docs.python.org/3/reference/simple_stmts.html#the-assert-statement)
* [The dangers of assert in Python](https://snyk.io/blog/the-dangers-of-assert-in-python/)
* [Feature: Python assert should be consider harmful](https://community.sonarsource.com/t/feature-python-assert-should-be-consider-harmful/38501) But note that Sonar did not implement this check.
