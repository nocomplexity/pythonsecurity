# Insecure Hashing

## Security Concerns

The use of insecure hashing functions in Python represents a critical cryptographic weakness. While the `hashlib` library provides robust cryptographic tools, it still allows the use of fundamentally broken algorithms like MD5 and SHA-1, which should be avoided in security-sensitive contexts.

:::{attention} Never use MD5 or SHA-1 for security-critical operations. 
Use SHA-256/SHA-512 for integrity, Argon2id for passwords, and explicitly mark any non-security uses with `usedforsecurity=False`. The consequences of weak hashing can be catastrophic, from credential theft to complete system compromise.
:::

### Why MD5 and SHA-1 Are Insecure

Both MD5 and SHA-1 are considered cryptographically broken for security purposes:

- **Collision Attacks**: An attacker can find two different inputs that produce the same hash digest. For MD5, collisions can be generated in seconds on consumer hardware. For SHA-1, practical collision attacks (SHAttered) have been demonstrated at a cost of approximately $110,000 in cloud computing time.
- **Preimage Resistance Failure**: While still computationally difficult, both algorithms show weaknesses that make them less resistant to finding inputs that match a given hash.
- **Length Extension Attacks**: Both MD5 and SHA-1 are vulnerable to length extension attacks, where an attacker can compute a valid hash for a message consisting of a secret plus additional data without knowing the secret.

### Real-World Exploits and Risks

The dangers of using these algorithms are not theoretical:

- **Certificate Forgery**: SHA-1 collisions have been used to create fraudulent SSL/TLS certificates, enabling man-in-the-middle attacks.
- **File Integrity Compromise**: Attackers can create malicious files that have the same MD5 or SHA-1 hash as legitimate files, bypassing integrity checks.
- **Password Cracking**: MD5 and SHA-1 are fast algorithms, making them susceptible to brute-force and dictionary attacks. Modern GPU clusters can perform billions of MD5 hashes per second.
- **Data Breaches**: Leaked passwords hashed with MD5 or SHA-1 can be quickly cracked, exposing user credentials.

### When Insecure Hashing Is Used

Despite their known weaknesses, MD5 and SHA-1 appear in codebases for several reasons:

- **Legacy Compatibility**: Older systems or protocols that require these algorithms
- **Non-Security Contexts**: Checksums for duplicate detection or data integrity (not cryptographic)
- **Convenience**: The algorithms are fast and readily available

### Python's `usedforsecurity` Parameter

From Python 3.9 onward, hashlib constructors include a keyword-only `usedforsecurity` parameter with a default value of `True`. Setting it to `False` allows the use of insecure algorithms, but requires explicit acknowledgment:

```python
import hashlib

# This will raise a ValueError in Python 3.9+ unless usedforsecurity=False
md5_hash = hashlib.md5(b"data")  # ValueError in FIPS mode

# Explicit opt-out - indicates non-security use
md5_hash = hashlib.md5(b"data", usedforsecurity=False)
```

**Critical Warning**: Setting `usedforsecurity=False` explicitly acknowledges you are using the algorithm outside a security context (e.g., as a non-cryptographic checksum). This should never be done for password hashing, digital signatures, or any security-critical operation.

## Preventive Measures

### 1. Use Secure Hashing Algorithms

Replace MD5 and SHA-1 with cryptographically strong alternatives:

| Use Case | Recommended Algorithm |
|----------|----------------------|
| Password hashing | Argon2id, bcrypt, scrypt, or PBKDF2 |
| File integrity | SHA-256, SHA-384, or SHA-512 |
| Digital signatures | SHA-256, SHA-384, or SHA-512 |
| Checksums (non-security) | MD5, SHA-1 (with `usedforsecurity=False`) |

### 2. Implement Password Hashing Correctly

For password storage, never use plain hashing algorithms like SHA-256 directly. Instead, use dedicated password hashing functions:

```python
import hashlib
import os

# GOOD: Using a password hashing algorithm
from passlib.hash import argon2

# Argon2id is the current gold standard
hash = argon2.hash("user_password")

# Or using hashlib's PBKDF2
salt = os.urandom(32)
key = hashlib.pbkdf2_hmac(
    'sha256',           # Hash algorithm
    b'password',        # Password
    salt,               # Salt
    600000,             # Iterations (OWASP recommended minimum)
    dklen=32            # Desired key length
)
```

### 3. Use HMAC for Message Authentication

For message authentication, use HMAC with a secure hash algorithm:

```python
import hmac
import hashlib

# GOOD: HMAC with SHA-256
secret_key = b'supersecretkey'
message = b'Important data'
signature = hmac.new(secret_key, message, hashlib.sha256).hexdigest()
```

### 4. Explicitly Mark Non-Security Uses

If you must use MD5 or SHA-1 for non-security purposes (e.g., checksums for duplicate detection), explicitly mark them as such:

```python
import hashlib

# ACCEPTABLE: Explicitly marking as non-security use
# Used only for deduplication, not security-critical
checksum = hashlib.md5(b"file_content", usedforsecurity=False).hexdigest()

# BETTER: Consider using xxHash or other non-cryptographic hashes
# for performance-critical checksums
import xxhash
fast_checksum = xxhash.xxh64(b"file_content").hexdigest()
```

## Example

### Vulnerable Implementation

```python
import hashlib
import hmac

# VULNERABLE: Using MD5 for password hashing
def store_password_md5(password):
    hash_obj = hashlib.md5(password.encode())
    return hash_obj.hexdigest()  # Broken!

# VULNERABLE: Using SHA-1 for integrity checks
def verify_integrity_sha1(filename, expected_hash):
    with open(filename, 'rb') as f:
        content = f.read()
        actual_hash = hashlib.sha1(content).hexdigest()
        return actual_hash == expected_hash  # Weak!

# VULNERABLE: Using MD5 for HMAC
def create_hmac_md5(secret, message):
    return hmac.new(secret, message, hashlib.md5).hexdigest()  # Broken!
```

### Secure Implementation

```python
import hashlib
import hmac
import os
from passlib.hash import argon2  # Install: pip install passlib

# GOOD: Password hashing with Argon2id
def store_password_secure(password):
    # Argon2id is the winner of the Password Hashing Competition
    hash = argon2.hash(password)
    return hash

# GOOD: Integrity checks with SHA-256
def verify_integrity_secure(filename, expected_hash):
    BUF_SIZE = 65536  # 64KB chunks
    sha256 = hashlib.sha256()
    with open(filename, 'rb') as f:
        while chunk := f.read(BUF_SIZE):
            sha256.update(chunk)
    return sha256.hexdigest() == expected_hash

# GOOD: HMAC with SHA-256
def create_hmac_secure(secret, message):
    return hmac.new(secret, message, hashlib.sha256).hexdigest()

# ACCEPTABLE: Non-security checksum (explicitly marked)
def deduplicate_checksum(content):
    # Used only for duplicate detection, not security-critical
    return hashlib.md5(content, usedforsecurity=False).hexdigest()
```

## Discussion

### When Is It OK to Use MD5 or SHA-1?

The short answer is: **almost never for security purposes**. However, there are limited legitimate use cases:

1. **Non-cryptographic checksums** for duplicate detection (e.g., file deduplication)
2. **Integrity checks** in non-security-critical contexts (e.g., verifying downloads from trusted sources)
3. **Compatibility** with legacy systems that cannot be updated
4. **Educational or research** purposes

Even in these cases:
- Explicitly mark with `usedforsecurity=False`
- Document why the insecure algorithm is being used
- Consider using faster non-cryptographic hashes like xxHash or CityHash

### The `usedforsecurity` Parameter Caveat

While `usedforsecurity=False` provides a way to use insecure algorithms, it creates a risk:

- **False Sense of Security**: Developers might incorrectly believe that setting this flag somehow makes the algorithm secure
- **Accidental Misuse**: Code might be copied into security-critical contexts without updating the algorithm
- **Maintenance Overhead**: Future developers might not understand why the flag is set

**Recommendation**: When you see `usedforsecurity=False` in a code review, demand:
1. Justification for using an insecure algorithm
2. Documentation explaining the non-security use case
3. Consideration of whether a non-cryptographic hash would be more appropriate


## More Information


- [Python hashlib Documentation](https://docs.python.org/3/library/hashlib.html)
- [hashlib `usedforsecurity` Parameter](https://docs.python.org/3/library/hashlib.html#hashlib-usedforsecurity)


- [CWE-327: Use of a Broken or Risky Cryptographic Algorithm](https://cwe.mitre.org/data/definitions/327.html)
- [CWE-328: Use of Weak Hash](https://cwe.mitre.org/data/definitions/328.html)
- [CWE-759: Use of a One-Way Hash without a Salt](https://cwe.mitre.org/data/definitions/759.html)
- [CWE-916: Use of Password Hash With Insufficient Computational Effort](https://cwe.mitre.org/data/definitions/916.html)
