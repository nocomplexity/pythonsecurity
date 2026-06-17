# Base64 Statements

Python Code should not contain obfuscated content, particularly code that uses `base64` (and related encodings) for encoding or decoding data is always suspected.

The following calls can be an indication for obfuscated contain that might cause a security weakness.

* `base64.b64decode`
* `base64.b64encode`
* `base64.b85encode`
* `base64.z85decode`

## Security concerns

Obfuscation using Base64 is a **long-standing and simple technique** commonly employed to conceal malicious code in Python projects. It enables attackers to hide payloads that would otherwise be easily identified.

The use of obfuscated content is uncommon in well-structured, legitimate Python code and is therefore considered a strong indicator of potential security risks.

:::{tip}
It is strongly recommended that any code containing Base64 encoding/decoding be carefully reviewed before deployment to production. Good modern Python SAST scanners, like [**Python Code Audit**](https://github.com/nocomplexity/codeaudit) check for `base64` module usage in code.
:::

**Key red flags include:**
* `base64.b64decode` followed immediately by `exec()` or `eval()`
* Long Base64 strings embedded in Python scripts
* Constructs such as `exec(base64.b64decode(...))` from untrusted sources

### Common Malware Patterns

Base64 encoding patterns are frequently found in Python-based malware and droppers:

| Pattern              | Code Snippet                                      | Security Concern                              | 
|----------------------|---------------------------------------------------|-----------------------------------------------|
| Standard b64 + exec | `exec(base64.b64decode(long_string))`            | Extremely common obfuscation technique          |
| Compressed           | `exec(zlib.decompress(base64.b64decode(...)))`   | Suggests larger hidden payload and evasion     |
| Multi-layer          | `base64.b64decode(base64.b64decode(...))`        | Attempts to bypass simple pattern matching     |
| Bytes decode         | `exec(base64.b64decode(data).decode())`          | Hides intent by decoding to string             |
| Using aliases        | `b64 = base64.b64decode; exec(b64(payload))`     | Evasion of basic static analysis               |
| Z85 / b85            | `base64.b85decode(...)` or `base64.z85decode(...)` | Non-standard encodings often indicate stealth|

### Security Considerations

Base encoding does not provide confidentiality. As noted in RFC 4648 (Section 12), care must be taken when implementing base encoding and decoding to avoid introducing vulnerabilities.

Security considerations section from RFC 4648 (section 12):

```text
Security Considerations

   When base encoding and decoding is implemented, care should be taken
   not to introduce vulnerabilities to buffer overflow attacks, or other
   attacks on the implementation.  A decoder should not break on invalid
   input including, e.g., embedded NUL characters (ASCII 0).

   If non-alphabet characters are ignored, instead of causing rejection
   of the entire encoding (as recommended), a covert channel that can be
   used to "leak" information is made possible.  The ignored characters
   could also be used for other nefarious purposes, such as to avoid a
   string equality comparison or to trigger implementation bugs.  The
   implications of ignoring non-alphabet characters should be understood
   in applications that do not follow the recommended practice.
   Similarly, when the base 16 and base 32 alphabets are handled case
   insensitively, alteration of case can be used to leak information or
   make string equality comparisons fail.

   When padding is used, there are some non-significant bits that
   warrant security concerns, as they may be abused to leak information
   or used to bypass string equality comparisons or to trigger
   implementation problems.

   Base encoding visually hides otherwise easily recognized information,
   such as passwords, but does not provide any computational
   confidentiality.  This has been known to cause security incidents
   when, e.g., a user reports details of a network protocol exchange
   (perhaps to illustrate some other problem) and accidentally reveals
   the password because she is unaware that the base encoding does not
   protect the password.

   Base encoding adds no entropy to the plaintext, but it does increase
   the amount of plaintext available and provide a signature for
   cryptanalysis in the form of a characteristic probability
   distribution.
```

## Example

```python
import base64
safe_string=b'aW1wb3J0IG9zOyBvcy5zeXN0ZW0oJ21hbGljaW91cyBjb21tYW5kJyk='

# Decoding patterns - still common in Python malware
b64 = base64.b64decode
exec(b64(safe_string))  
```

What some Python novice did - the attacker side:

```python
payload = b"import os; os.system('malicious command')"
encoded_b64 = base64.b64encode(payload)

```

## References

* [Python Documentation – base64](https://docs.python.org/3/library/base64.html)
* [RFC 4648 – Security Considerations](https://datatracker.ietf.org/doc/html/rfc4648#section-12)
* [Base64 Malleability in Practice](https://eprint.iacr.org/2022/361.pdf)

