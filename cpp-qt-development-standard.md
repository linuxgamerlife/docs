<div align="center">

# C++ / Qt Secure Development Standard

Engineering standards for building safe, maintainable, and secure C++ / Qt applications.

<br>

![C++](https://img.shields.io/badge/C++-20-blue?logo=cplusplus&logoColor=white)
![Qt](https://img.shields.io/badge/Qt-6-green?logo=qt&logoColor=white)
![CMake](https://img.shields.io/badge/CMake-build-blue?logo=cmake&logoColor=white)

<br>

Standards designed to be **clear for humans** and **deterministic for AI-assisted development**.

</div>

---

# Purpose

This document defines the engineering, safety, and security standards required for all C++ and Qt code produced for this project.

These rules exist to prevent common classes of problems including:

- memory corruption
- undefined behavior
- race conditions
- threading mistakes
- security vulnerabilities
- unclear ownership
- fragile architecture

All code must follow these standards unless an explicit exception is approved and documented.

The goal is to produce code that is **safe, maintainable, testable, predictable, and secure**.

---

# Language and Core Standards

All code must use modern C++.

- Preferred language standard: **C++20**
- Minimum allowed standard: **C++17**

Do not introduce legacy C++ patterns when modern equivalents exist.

Code must follow:

- **C++ Core Guidelines**
- **SEI CERT C++ Secure Coding Standard**
- **Qt Official Best Practices**

### Key expectations

- Use **RAII** for resource management
- Prefer **stack allocation** where possible
- Use **standard containers and standard library features**
- Avoid **undefined behavior**
- Prefer **clarity over cleverness**
- Favor **explicit ownership and lifetime management**
- Use **const correctness aggressively**
- Prefer **strong types** where appropriate

---

# Memory and Ownership Rules

Memory safety is critical.

### Required practices

- Use **RAII everywhere**
- Prefer **automatic storage duration**
- Use **smart pointers for ownership management**
- `std::unique_ptr` is the **default owning pointer**
- `std::shared_ptr` may only be used when shared ownership is required
- Shared ownership must be **documented**
- Avoid raw `new` and `delete`
- Never use **owning raw pointers**
- Make ownership **explicit in interfaces**
- Avoid **cyclic ownership**

### Prevent the following bug classes

- use-after-free
- double free
- dangling pointers
- invalid references
- leaked resources

### QObject Ownership Rules

For `QObject`-derived classes:

- Use **Qt parent-child ownership deliberately**
- Never manually delete a `QObject` that has a parent
- Do not combine **Qt parent ownership** with **smart pointer ownership**
- The owner must be **obvious at the creation site**
- Lifetime must be **clear and deterministic**

---

# Undefined Behavior Prevention

The following practices are prohibited:

- C-style casts
- unsafe `reinterpret_cast`
- unbounded array access
- unchecked pointer arithmetic
- use of uninitialized memory
- reliance on signed integer overflow
- unsafe narrowing conversions

Preferred alternatives:

- `static_cast`
- safe container access
- explicit initialization
- explicit bounds checking

Always validate assumptions and handle edge cases explicitly.

---

# Error Handling

All failures must be handled intentionally.

### Rules

- Never silently ignore errors
- Check return values
- Provide clear error propagation
- Ensure failure modes are safe
- Log failures without exposing secrets

### Error Handling Model

- Exceptions must **not** be used for normal application flow
- Recoverable errors must use a **result-based model**
- Prefer `std::expected<T, Error>` where available
- Do not mix error handling models within a subsystem
- Convert low-level errors into safe user-facing messages

---

# Input Validation

All external input must be treated as **untrusted**.

This includes:

- UI input
- configuration files
- command line arguments
- files and file paths
- network data
- IPC messages
- database input
- environment variables

### Rules

- Validate all input before use
- Reject malformed data early
- Apply size limits where appropriate
- Avoid dangerous implicit conversions
- Validate enums explicitly
- Validate filesystem paths before use

---

# Qt-Specific Rules

Qt introduces its own ownership and threading model which must be respected.

## QObject Lifetime

- Use Qt parent-child ownership deliberately
- Do not manually delete parent-owned QObjects
- Ownership must remain simple and obvious

## Threading

- Never access `QObject` from the wrong thread
- Use signals and slots for cross-thread communication
- Use queued connections across threads
- Make thread affinity explicit
- Protect shared state appropriately

## UI Thread Rules

Never block the UI thread with:

- file I/O
- network I/O
- long computations
- sleep calls
- blocking waits

Heavy work must run in worker threads.

## Networking

- Use Qt networking APIs safely
- Validate remote data
- Enforce TLS verification
- Implement timeouts
- Handle malformed responses safely

## Database Access

- Use **parameterized SQL queries**
- Never build SQL queries via string concatenation
- Handle transactions and failures explicitly

---

# File System Safety

File operations must be secure.

Required practices:

- Validate file paths
- Canonicalize paths when necessary
- Prevent path traversal
- Avoid unsafe temporary file creation
- Validate permissions where required
- Handle filesystem errors explicitly

---

# Network Security

All network communication must assume hostile conditions.

### Requirements

- Use TLS for remote communication
- Validate certificates
- Do not disable certificate verification in production
- Sanitize remote input
- Enforce timeouts
- Handle partial responses safely

---

# Secret and Credential Handling

Secrets must **never be embedded directly in source code**.

Do not hardcode:

- API keys
- tokens
- passwords
- certificates
- private keys
- client secrets

### Rules

- Use secure platform storage where required
- Do not expose secrets to logs
- Mask secrets in diagnostics
- Avoid exposing secrets to UI or debug tools

---

# Logging Rules

Logs must assist debugging without exposing sensitive information.

Logs must never contain:

- passwords
- API keys
- tokens
- private user data
- cryptographic secrets

Security failures must be logged carefully without leaking sensitive data.

---

# Code Quality Requirements

All code must compile cleanly with strict warnings.

### Rules

- Warnings must be treated as **errors**
- The project must build cleanly on:

  - **GCC**
  - **Clang**

- Static analysis tools must pass

### Required tools

- `clang-tidy`
- `cppcheck`

### Recommended clang-tidy rule groups

- modernize
- bugprone
- performance
- readability
- cppcoreguidelines

Prefer **clear code over clever code**.

---

# Formatting and Style Requirements

Formatting must be consistent across the entire codebase.

### Rules

- Use **clang-format** with a checked-in configuration
- Follow consistent naming conventions
- Prefer descriptive identifiers
- Keep functions reasonably small
- Avoid style inconsistencies between generated and existing code
- Comment **intent and invariants**, not obvious syntax

---

# Runtime Safety Tools

Debug and test builds must run with sanitizers enabled where possible.

### Required

- AddressSanitizer
- UndefinedBehaviorSanitizer

### Recommended

- ThreadSanitizer

Sanitizer issues must be **fixed**, not ignored.

---

# Testing Requirements

Testing is mandatory for core logic.

### Frameworks

- **Qt Test (QTest)**
- **QSignalSpy**
- **QAbstractItemModelTester**
- **Qt Quick Test** if QML is used

### Required coverage

- business logic
- file handling
- configuration loading
- networking
- database access
- error paths
- malformed input
- timeout conditions

Every significant bug fix must include a **regression test**.

Tests must remain **deterministic**.

---

# Fuzz Testing

Input parsing code should be fuzz tested where practical.

Targets include:

- file parsers
- protocol handlers
- configuration readers
- import/export code
- deserialization logic

Fuzz testing should focus on **robustness and crash resistance**.

---

# Architecture Guidelines

System design must remain clear and testable.

Recommended structure:

- Separate UI from business logic
- Ensure business logic is testable without the GUI
- Avoid global state
- Avoid unnecessary singletons
- Document trust boundaries
- Document thread boundaries
- Document ownership boundaries

Prefer **simple designs over clever abstractions**.

---

# Feature Implementation Expectations

When implementing new features:

1. Describe the design
2. Implement the code
3. Explain ownership and lifetime
4. Explain thread behavior
5. Explain error handling
6. Describe failure modes
7. Provide tests

For AI-assisted development, every feature must make the following clear:

- ownership
- lifetime
- thread affinity
- failure behavior
- trust boundaries

---

# Engineering Philosophy

This project prioritizes:

- safety
- clarity
- correctness

Fast iteration is welcome, but **unsafe shortcuts are not acceptable**.

If a quick solution introduces safety or security risks, the safer design must be implemented first unless explicitly approved.

Every class and function must clearly communicate:

- ownership
- lifetime
- thread affinity
- failure behavior

---

# Summary

This project enforces a **safety-first development approach** using:

- modern C++
- secure coding standards
- Qt best practices
- static analysis
- runtime sanitizers
- comprehensive testing

These rules exist to prevent common classes of bugs and vulnerabilities and to ensure the codebase remains reliable and maintainable.
