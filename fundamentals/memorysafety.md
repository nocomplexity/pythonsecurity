---
title: Memory Safety
short_title: Memory Safety
---

:::{attention} Importance of memory safety for software
:class: dropdown
Memory safety is a foundational property in programming languages that prevents programmers from making mistakes when managing computer memory. When a programming language is memory-safe, the language runtime or compiler automatically guards against bugs where a program accesses, modifies, or frees memory it shouldn't.
:::

Many cybersecurity vulnerabilities in software stem from a class of defects commonly known as memory safety issues or buffer overflows. When software mishandles memory during execution, it can lead to application crashes or create openings that malicious actors can exploit to compromise the system.

A primary defence against these flaws is the use of a memory-safe programming language. In a memory-safe language, manual memory management—such as explicit allocation and deallocation—is abstracted away and handled automatically by the language's runtime environment or interpreter rather than the developer. This significantly reduces the likelihood of introducing critical memory-related bugs.

* Memory-safe languages include: Rust, Go, C#, Java, Swift, Python, and JavaScript.
* Memory-unsafe languages include: C, C++, Fortran and Assembly.

Python is a language with built-in memory management (such as automatic garbage collection).

By default you do not interact directly with raw memory using Python code. **So Python is considered a memory-safe language by default.** This shields developers from entire categories of common weaknesses like buffer overflows. 

:::{note} CPython can have bugs
:class: dropdown
Since 100% security does not exist and AI and humans do make mistakes:
In rare occasions CPython can have bugs that relate to memory management.

CPython (the standard and most widely used implementation of Python) is written in C. Because C is a memory-unsafe language, the underlying interpreter codebase is susceptible to traditional memory bugs like buffer overflows, use-after-free errors, and reference-counting mistakes.

Despite rigorous code reviews, automated testing, and fuzzing, Python's core developers (who are human) occasionally introduce or miss subtle memory corruption bugs.
:::

However, relying on a memory-safe language like Python is only one component of a comprehensive application security strategy. While Python protects against memory-related exploits, it does not prevent logic flaws, injection attacks, insecure data handling, or other non-memory vulnerabilities.

