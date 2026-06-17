# Pickle Function

:::{danger} 
Unpickling will import any class or function that it finds in the pickle data. This is a severe security concern as it permits the unpickler to import and invoke arbitrary code. 

**Never use `pickle.load()`, `pickle.loads()` or `pickle.Unpickler` on data received from an untrusted or unauthenticated source.**
:::

## Security Concerns

The security concern with the `pickle` module in Python revolves around **deserialisation of untrusted data**. When you use `pickle.load()` to deserialise a byte stream, the `pickle` module reconstructs Python objects from that stream by importing any class or function it encounters. A malicious attacker can craft a pickled payload that, when deserialised, can:

1. **Execute arbitrary code:** The `pickle` protocol can be manipulated to cause the deserialiser to import arbitrary modules and call arbitrary functions with arbitrary arguments. This permits an attacker to execute system commands, delete files, or perform any action the Python process running `pickle.load()` has permissions to carry out. This is commonly referred to as a "deserialisation vulnerability" or "arbitrary code execution."

2. **Cause Denial of Service (DoS):** An attacker could create a pickled object that, when deserialised, consumes excessive memory or CPU resources, leading to your application crashing or becoming unresponsive.

## Preventive Measures

The most effective preventive measure is to **avoid using `pickle` for deserialising untrusted data entirely**. Where this is not possible, consider the following approaches:

- **Use alternative serialisation formats:** Consider using `json`, `msgpack`, or `protobuf` for data exchange with untrusted sources. These formats are limited to data structures and do not support arbitrary code execution.
  
- **Implement cryptographic signing:** If you must use `pickle`, sign your pickled data with a cryptographic signature (e.g., using `hmac`) before transmission and verify the signature before deserialisation. This ensures the data has not been tampered with.

- **Run deserialisation in a sandbox:** Isolate the unpickling process in a restricted environment, such as a container or a separate process with limited system privileges.

- **Use `pickle.restricted` (Python 3.x):** The `pickle` module provides a `restricted` attribute that can be used to limit the classes and functions that can be imported during unpickling. However, this is not a complete security solution and should be used with caution.

## Example

Consider the following malicious pickled payload:

```python
import pickle
import os

class Malicious:
    def __reduce__(self):
        return (os.system, ('rm -rf /',))

malicious_data = pickle.dumps(Malicious())
pickle.loads(malicious_data)  # Executes 'rm -rf /'
```

In this example, the `__reduce__` method returns a tuple instructing the unpickler to call `os.system` with the argument `'rm -rf /'`. When deserialised, this would execute a destructive system command.

## Discussion

The root cause of the security issue lies in Python's dynamic nature: `pickle` was designed for flexibility, allowing serialisation of complex Python objects, including functions and classes. This flexibility comes at the cost of security when handling untrusted data.

The security weakness is not in `pickle` itself but in how it is used. The Python documentation explicitly warns against using `pickle` with untrusted data. 


## More Information

- [Python `pickle` Documentation - Security Considerations](https://docs.python.org/3/library/pickle.html#pickle-restrict)
