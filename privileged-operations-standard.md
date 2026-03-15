# Privileged Operations Standard

## Purpose

This standard defines how applications that perform privileged operations must be designed and implemented.

The goal is to minimise risk when interacting with system-level functionality such as:

- package installation
- repository changes
- device configuration
- filesystem modifications

These rules apply to any component capable of performing operations requiring administrative privileges.


## Core Principles

| Principle | Description |
|---|---|
| Minimal Privilege | Only the smallest amount of code should run with elevated privileges |
| Explicit Boundaries | Privilege transitions must be clearly defined and documented |
| Untrusted Input | All requests crossing the privilege boundary must be treated as untrusted input |


## Architecture Requirements

### GUI Privileges

**Rule**

User interfaces must never run as root.

GUI code must run as the normal user and request privileged operations when required.


### Privileged Isolation

**Rule**

Privileged functionality must be implemented in a dedicated helper or service responsible only for administrative tasks.

The majority of application code must remain unprivileged.


### Allow-Listed Operations

Privileged helpers must only support predefined operations.

The following must never be allowed:

- arbitrary command execution
- arbitrary filesystem paths
- arbitrary parameters


### Shell Execution

Shell execution should be avoided.

Programs must be executed directly using explicit program and argument lists.

Example:

```
program: dnf
args: install package-name
```


### Input Validation

Inputs used in privileged commands must be validated.

Validation must ensure:

- expected format
- permitted character set
- expected value range


## Failure Handling

### Fail Safe Behaviour

Privileged operations must fail safely.

Unexpected errors or exit codes must stop execution rather than continuing in a partially configured state.


### Idempotent Behaviour

Where possible, operations should be safe to run multiple times without corrupting system state.


## Logging

Privileged operations must produce logs including:

- operation requested
- operation executed
- result or failure condition

Logs must never expose sensitive information.


## Compliance Checklist

Before merging code that performs privileged operations:

- [ ] GUI does not run as root
- [ ] Privileged functionality is isolated
- [ ] Commands are allow-listed
- [ ] Shell execution is avoided
- [ ] Inputs are validated
- [ ] Fail-safe behaviour implemented
- [ ] Logging present
