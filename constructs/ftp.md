# FTP Statement

The use of `ftplib.FTP` from Python's standard library is **insecure and should be avoided** in modern application development. This class implements the legacy File Transfer Protocol (FTP), which lacks native encryption for both authentication and data transmission.

:::{danger} 
Never ever allow or use `ftplib.FTP`.

Even within isolated or private networks, the risk of credential theft and data interception is unacceptably high. Always implement protocols that secure data transfers by default.
:::

## Security Concerns

Using `ftplib.FTP` introduces several critical vulnerabilities that compromise the confidentiality and integrity of your data:

### Cleartext Credentials
Usernames and passwords are transmitted in plain text over the network, making them trivial to intercept via packet sniffing tools such as Wireshark or tcpdump.

### Unencrypted Data Transfer
File contents, directory structures, and command responses are sent without encryption. This exposes sensitive information to eavesdropping and allows attackers to read or modify data in transit.

### Man-in-the-Middle (MITM) Attacks
Because FTP lacks cryptographic integrity checks, an attacker positioned between the client and server can capture, alter, or inject malicious content into the communication stream undetected.

### Deceptive Aliasing
Renaming the class (e.g., `from ftplib import FTP as SecureFTP`) provides no technical security whatsoever and is **dangerously misleading**. This practice can create a false sense of security among developers and should be strictly prohibited in code reviews.

### Firewall and NAT Complications
FTP's use of multiple ports (control and data channels) frequently leads to firewall and Network Address Translation (NAT) traversal problems, increasing operational complexity and security risk.

## Preventive Measures

To mitigate the risks associated with file transfer operations, adopt the following security practices:

### Avoid `ftplib.FTP` Entirely
Never use plain FTP in production environments or for handling sensitive data, regardless of network segmentation or perceived isolation.

### Use Secure Alternatives
- **Prefer SFTP** (SSH File Transfer Protocol) using the `paramiko` library, which provides strong encryption and authentication through the SSH protocol.
- **Alternatively, use FTPS** (FTP over TLS/SSL) via `ftplib.FTP_TLS` where SFTP is not available. Ensure that explicit TLS is enabled and certificate validation is enforced.

### Enforce Encrypted Communication
Verify that all authentication credentials and file transfers occur exclusively over encrypted channels. Reject any connection that does not support modern cryptographic standards.

### Validate Third-Party Integrations
Before integrating external services, confirm that they support secure protocols (SFTP or FTPS) and are configured to enforce encryption.

### Apply Secure Credential Handling
Store and manage credentials using secure mechanisms such as environment variables, dedicated secrets managers (e.g., HashiCorp Vault, AWS Secrets Manager), or encrypted configuration files. Never hard-code credentials in source code.

## Example

The following example demonstrates both insecure and secure approaches to file transfer using only Python's standard library:

**INSECURE: Do not use this approach**
```python
from ftplib import FTP

ftp = FTP("example.com")
ftp.login("username", "password")  # Credentials sent in cleartext
with open("file.txt", "rb") as f:
    ftp.storbinary("STOR file.txt", f)
ftp.quit()
```

**SECURE: Use FTPS with explicit TLS (standard library only)**

```python
from ftplib import FTP_TLS
import ssl

# Create FTPS connection with explicit TLS
ftps = FTP_TLS("example.com")
ftps.login("username", "password")

# Force encryption for the data channel
ftps.prot_p()

# Optional: Enforce strict certificate validation
ftps.ssl_context = ssl.create_default_context()
ftps.ssl_context.check_hostname = True
ftps.ssl_context.verify_mode = ssl.CERT_REQUIRED

# Transfer file securely
with open("file.txt", "rb") as f:
    ftps.storbinary("STOR file.txt", f)

ftps.quit()
```

### FTPS Configuration Best Practices

When using `ftplib.FTP_TLS`, implement these additional security measures:

```python
import ssl
from ftplib import FTP_TLS

# Create a secure SSL context
context = ssl.create_default_context()
context.set_ciphers("ECDHE+AESGCM:ECDHE+CHACHA20:DHE+AESGCM:DHE+CHACHA20:!aNULL:!MD5:!DSS")
context.options |= ssl.OP_NO_SSLv2 | ssl.OP_NO_SSLv3 | ssl.OP_NO_TLSv1 | ssl.OP_NO_TLSv1_1

# Establish secure connection
ftps = FTP_TLS("example.com", context=context)
ftps.login("username", "password")
ftps.prot_p()  # Critical: Force encrypted data channel

# Verify server certificate matches hostname
ftps.ssl_context.check_hostname = True

# Transfer files with encryption
with open("sensitive_data.txt", "rb") as f:
    ftps.storbinary("STOR sensitive_data.txt", f)

# Secure logout
ftps.quit()
```

### Important FTPS Considerations

- **Always call `prot_p()`**: Without this, your data channel remains unencrypted even if the control channel is secure.
- **Certificate Validation**: Never disable certificate verification (avoid `ssl.CERT_NONE` in production).
- **TLS Version**: Require TLS 1.2 or higher; disable older, vulnerable protocols.
- **Cipher Suites**: Use only strong, modern cipher suites that provide forward secrecy.

## Discussion

The decision to avoid `ftplib.FTP` stems from fundamental protocol design flaws that cannot be mitigated through configuration or workarounds. While FTP remains in use due to legacy system requirements, modern security standards mandate encryption for all network communications.

When SFTP is unavailable, FTPS with explicit TLS provides an acceptable alternative, provided that certificate validation is enforced and weak cryptographic ciphers are disabled. However, SFTP is generally preferred due to its simpler firewall traversal (single port), better integration with SSH infrastructure, and more comprehensive security features.

Organizations should conduct regular audits to identify and remediate any lingering FTP usage. Good SAST scanning tools for Python detect imports of `ftplib.FTP` and flag them.

## More Information

- [CWE-319: Cleartext Transmission of Sensitive Information](https://cwe.mitre.org/data/definitions/319.html)
- [Python `ftplib` Documentation](https://docs.python.org/3/library/ftplib.html)
- [NIST SP 800-52: Guidelines for the Selection and Use of Transport Layer Security (TLS) Implementations](https://csrc.nist.gov/publications/detail/sp/800-52/rev-2/final)
