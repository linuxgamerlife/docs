# Privileged Operations Standard

Purpose

This standard defines how applications that perform privileged operations must be designed and implemented. The goal is to minimise risk when interacting with system-level functionality such as package installation, repository changes, device configuration, or filesystem modifications.

These rules apply to any component capable of performing operations requiring administrative privileges.


Principles

Privileged code must be minimal.

The amount of code running with elevated privileges must be as small as possible. The majority of the application must run as a normal user.

Privilege boundaries must be explicit.

Any transition from unprivileged code to privileged execution must be clearly defined and documented.

All requests crossing the privilege boundary must be treated as untrusted input.


Architecture Requirements

GUI components must never run as root.

User interfaces must run with normal user privileges.

Privileged functionality must be isolated.

System-level actions must be implemented in a dedicated helper or service responsible only for performing specific administrative tasks.

Privileged operations must be allow-listed.

Only predefined operations may be executed. Arbitrary commands or parameters must never be accepted.

Shell execution must be avoided.

Privileged operations must invoke programs directly with explicit argument lists. Shell execution such as `bash -c` must not be used unless absolutely required and must never include user-controlled data.

All inputs must be validated.

Any input used in privileged commands must be validated against strict rules including allowed characters, formats, and expected values.


Failure Handling

Privileged operations must fail safely.

Unexpected exit codes or errors must stop execution rather than continuing in a partially configured state.

Operations must be idempotent where possible.

Repeated execution should not corrupt the system or produce inconsistent states.


Logging

Privileged actions must generate clear logs indicating:

- Operation requested
- Operation executed
- Result or error condition

Logs must never expose secrets or sensitive user data.


Review Requirements

All privileged code must receive additional security review before merging.

Changes affecting privileged operations must document:

- What operation is performed
- What inputs are accepted
- How those inputs are validated
- What failure behaviour is expected
