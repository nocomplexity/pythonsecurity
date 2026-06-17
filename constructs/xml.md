# XML Security

:::{danger}
XML parsing in Python can expose applications to severe security vulnerabilities, including denial of service attacks, local file access, and firewall circumvention. The `xmlrpc` module is particularly vulnerable to the "decompression bomb" attack.

**Always implement security measures when parsing XML from untrusted sources. Never parse untrusted XML without proper safeguards in place.**
:::

## Security Concerns

XML processing in Python presents several significant security risks that must be understood before parsing any untrusted XML data:

1. **Denial of Service (DoS) Attacks:** Attackers can exploit XML features to overwhelm your application:
   - **Decompression bombs** (also known as "billion laughs" attacks): Nested entity expansions that consume exponential amounts of memory, causing the application to crash or become unresponsive.
   - **Large file uploads**: Extremely large XML files that exhaust system resources.
   - **Deep nesting**: Excessively nested XML structures that cause stack overflow or excessive CPU consumption.

2. **External Entity (XXE) Attacks:** By defining external entities, an attacker can:
   - Read arbitrary local files (e.g., `/etc/passwd`, configuration files).
   - Generate network connections to internal or external machines, potentially bypassing firewalls.
   - Perform Server-Side Request Forgery (SSRF) attacks.
   - Cause denial of service through resource exhaustion.

3. **RCE and File Inclusion:** In misconfigured parsers, attackers may be able to include external files that contain executable code or cause arbitrary file writes.

The Python `xml` module provides great benefits for legitimate XML processing, but you must take security concerns very seriously and implement appropriate safeguards.

## Preventive Measures

To securely parse XML in Python, adopt the following preventive measures:

- **Use secure parser configurations:** Disable external entity processing and DTD (Document Type Definition) loading in your XML parser. For example, when using `xml.etree.ElementTree`:
  ```python
  from xml.etree.ElementTree import XMLParser
  parser = XMLParser(disable_dtd=True)
  ```

- **Set resource limits:** Restrict the maximum size of XML documents and limit entity expansion to prevent decompression bombs:
  ```python
  from xml.etree.ElementTree import XMLParser, fromstring
  from defusedxml.ElementTree import fromstring as safe_fromstring
  ```

- **Use `defusedxml` library:** The `defusedxml` library provides safe versions of all Python's standard library XML parsers, with protections against all known XML attacks:
  ```bash
  pip install defusedxml
  ```

- **Validate XML schemas:** Validate untrusted XML against a strict schema (XSD) before processing to ensure it conforms to expected structure and content.

- **Implement timeout and rate limiting:** Apply timeout mechanisms to XML parsing operations and rate-limit XML requests to prevent resource exhaustion.

- **Avoid using `xmlrpc` with untrusted endpoints:** The `xmlrpc` module is particularly vulnerable. Use it only with trusted services, or switch to more secure alternatives like JSON-RPC with proper validation.

## Example

Consider this vulnerable XML parser that is susceptible to a billion laughs attack:

```python
"""Vulnerable XML parser - DO NOT USE"""
import xml.etree.ElementTree as ET

def parse_vulnerable(xml_data):
    # This is vulnerable to decompression bomb attacks
    tree = ET.fromstring(xml_data)
    return tree

# Malicious payload - billion laughs attack
billion_laughs = """<?xml version="1.0"?>
<!DOCTYPE lolz [
  <!ENTITY lol "lol">
  <!ENTITY lol1 "&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;&lol;">
  <!ENTITY lol2 "&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;&lol1;">
  <!ENTITY lol3 "&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;&lol2;">
]>
<root>&lol3;</root>"""
```

To securely parse the same XML, use `defusedxml`:

```python
"""Secure XML parser using defusedxml"""
from defusedxml.ElementTree import fromstring

def parse_secure(xml_data):
    # defusedxml protects against all known XML attacks
    tree = fromstring(xml_data)
    return tree

# This will safely handle the billion laughs attack
parsed_tree = parse_secure(billion_laughs)  # Raises appropriate exception
```

Alternatively, if you must use the standard library, configure it securely:

```python
"""Secure XML parser using standard library with safeguards"""
from xml.etree.ElementTree import XMLParser

def parse_with_safeguards(xml_data):
    parser = XMLParser(disable_dtd=True)  # Disable DTD to prevent many attacks
    tree = parser.feed(xml_data)
    return tree
```

## Discussion

The security issues with XML parsing stem from XML's powerful features—DTDs, entities, external references—which were designed for flexibility and interoperability but can be weaponised against applications. This is sometimes referred to as "XML is not secure by default."

The Python standard library's XML parsers are **not** secure by default. Each parser has different default configurations and vulnerabilities. This inconsistency has led to the creation of `defusedxml`, which provides a consistent, secure interface to all standard XML parsers.

A common misconception is that using `xml.etree.ElementTree` is sufficient for security. While it is safer than older parsers like `xml.sax` or `xml.dom`, it still permits DTD processing and entity expansion by default in older Python versions. Always explicitly disable DTD and external entity processing.

For modern web applications, consider using JSON instead of XML where possible. JSON is less complex, has fewer attack vectors, and is generally more straightforward to parse securely. However, if XML is required for interoperability, `defusedxml` should be your default choice.

## More Information

- [Python XML Processing Security Documentation](https://docs.python.org/3/library/xml.html#xml-security)
- [Billion Laughs Attack - Wikipedia](https://en.wikipedia.org/wiki/Billion_laughs_attack)
- [CWE-611: Improper Restriction of XML External Entity Reference](https://cwe.mitre.org/data/definitions/611.html)
- [CWE-776: Improper Restriction of Recursive Entity References](https://cwe.mitre.org/data/definitions/776.html)
- [Python XML Library Documentation](https://docs.python.org/3/library/xml.html)
