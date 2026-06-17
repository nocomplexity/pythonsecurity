# HTTP Server Use

## Security Concerns

The Python standard library provides `http.server` as a convenient module for quickly launching a web server. However, this convenience comes with significant security implications that should be considering before using.

:::{attention}
The `http.server` module is **explicitly designed for development and testing purposes only**. 
:::

The `http.server` module implements **only** a basic HTTP server that lacks essential security features required for production environments, including:
- **No encryption**: Communication occurs over plain HTTP, making all data transmissions vulnerable to interception and eavesdropping
- **No authentication mechanisms**: The server provides no built-in user authentication or authorisation controls
- **Limited request handling**: The server is susceptible to various denial-of-service attacks and malformed request exploits
- **Directory traversal vulnerabilities**: Without careful configuration, the server may expose files outside the intended document root
- **No input sanitisation**: The server does not validate or sanitise incoming requests, potentially leading to injection attacks

These vulnerabilities make `http.server` unsuitable for **any** environment where data confidentiality, integrity, or availability is a concern.

## Preventive Measures

### Development Environment Isolation

The `http.server` module **MUST** be confined exclusively to local development environments. Under no circumstances should this server be exposed to untrusted networks, including local area networks (LANs) or virtual private networks (VPNs) that contain other users.

### Production Deployment Alternatives

For production deployments, Python applications **SHOULD** be served through industry-standard web servers that provide robust security features:

- **Apache HTTP Server**: Offers comprehensive security modules, authentication mechanisms, and SSL/TLS support
- **NGINX**: Provides high-performance request handling with built-in security features and rate limiting
- **Gunicorn or uWSGI**: Application servers designed to work behind reverse proxies, providing process isolation and security boundaries

When deploying Python web applications, adopt a defence-in-depth approach by placing your application behind a production-grade web server that handles security-critical functions.

### Container and Deployment Considerations

Even within containerised environments (Docker, Kubernetes, etc.), using `http.server` remains inadvisable. Container isolation does not compensate for the fundamental security deficiencies of this module. The principle of least privilege dictates that production services should use purpose-built web servers with proven security track records.

## Example

The following example demonstrates the **incorrect** usage of `http.server` in a development scenario that could accidentally expose sensitive data:

This is for demonstration of insecure practice only!

DO NOT use this code in production or on any network!

```python

from http.server import HTTPServer, SimpleHTTPRequestHandler
import os

# This serves the current directory over HTTP on all interfaces
# This is extremely dangerous and should never be used
server_address = ('0.0.0.0', 8000)
httpd = HTTPServer(server_address, SimpleHTTPRequestHandler)

# This will serve all files in the current directory
# Potentially exposing source code, configuration files, and secrets
print("Serving files from:", os.getcwd())
print("WARNING: This server is insecure and should only be used for local testing")
httpd.serve_forever()
```


## Discussion

A valid question is: Why Is This Module Still in the Standard Library?

The `http.server` module remains in the Python standard library due to its utility for:
- Quick local testing of static content
- Educational purposes to demonstrate HTTP fundamentals
- Internal tooling in isolated development environments

However, its inclusion should **never** be interpreted as an endorsement for production use. The module's documentation explicitly warns against production deployment.

A common misconception is that development environments are inherently secure because they are "not production." This thinking is dangerous because:

1. Development environments often contain production-like data or connection strings
2. Many security breaches originate from compromised development systems
3. The boundary between development and production can become blurred in CI/CD pipelines
4. Internal networks are frequently targeted by attackers seeking footholds

Treat development environments with the same security rigour as production systems. If a server is accessible on any network interface other than `localhost`, it should be considered at risk.


## More Information

- [Python `http.server` Module Documentation](https://docs.python.org/3/library/http.server.html#module-http.server)
- [Python `http.server` Security Considerations](https://docs.python.org/3/library/http.server.html#http-server-security)


