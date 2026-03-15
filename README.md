# Development Standards

## Important Notice

I am not a professional software developer. The projects I create rely heavily on generative AI to assist with code development and production.

All code is reviewed using automated tooling and security analysis where possible, and applications are tested extensively as I go. I also perform ongoing testing and security analysis as projects evolve.

Despite these precautions, software in this ecosystem may still contain bugs, security issues, or unintended behaviour.

Any use of software from these repositories is done entirely at the user's own risk.

All software is provided under the terms of the MIT licence and is distributed **"as is" without warranty of any kind**.


## Purpose

This repository contains the engineering and security standards used across LinuxGamerLife projects.

These standards exist to ensure that software produced for the project is developed in a responsible, transparent, and structured way, even when AI-assisted development is involved.

They are designed to provide guardrails around:

- code quality
- security architecture
- privileged operations
- release integrity
- AI-assisted development


## Standards Overview

### C++ / Qt Development Standard

Defines the engineering rules for C++ and Qt applications.

This standard focuses on:

- modern C++ usage
- memory safety
- RAII and resource management
- Qt best practices
- static analysis
- sanitizers
- defensive coding

File: `cpp-qt-development-standard.md`


### Privileged Operations Standard

Defines how applications that perform system-level operations must be designed and implemented.

This standard is especially important for tools that:

- install packages
- modify system configuration
- enable repositories
- interact with hardware
- perform administrative actions

It enforces:

- strict privilege separation
- minimal root-level code
- explicit allow-listed operations
- validated inputs
- safe failure behaviour

File: `privileged-operations-standard.md`


### Secure Design Review Standard

Defines the process used to evaluate security risks during the design phase before implementation begins.

The goal is to identify potential problems early and design systems safely from the start.

This includes:

- identifying trust boundaries
- understanding what assets a feature affects
- evaluating misuse scenarios
- applying least privilege principles

File: `secure-design-review-standard.md`


### Release Integrity Standard

Defines how releases must be produced and verified.

Because some LinuxGamerLife tools interact with system configuration, users must be able to trust the binaries they install.

This standard covers:

- controlled build environments
- CI-generated release artifacts
- cryptographic signing
- checksums for distributed binaries
- dependency control
- vulnerability response procedures

File: `release-integrity-standard.md`


### AI-Assisted Development Safety Standard

Defines how generative AI may be used responsibly during development.

AI is treated as a productivity tool, not a trusted source of production-ready code.

Code generated with AI must pass automated verification using reputable tooling such as:

- static analysis
- CodeQL
- compiler warnings
- sanitizers
- automated testing

High or critical issues identified by these tools must be investigated and addressed before release.

File: `ai-assisted-development-safety-standard.md`


## How These Standards Work Together

These standards form a layered development model.

1. Features are reviewed using secure design principles.
2. Code must follow strict C++ and Qt engineering standards.
3. Privileged operations are isolated and tightly controlled.
4. Automated tools verify security and reliability.
5. Releases are built and distributed with verifiable integrity.

This approach helps ensure that LinuxGamerLife projects remain transparent, testable, and as safe as reasonably possible.


## Scope

These standards apply to:

- software developed under the LinuxGamerLife project
- tools that interact with system configuration
- projects where AI-assisted development is used


## Licence

These standards are published under the MIT licence.

Software developed using these standards may be distributed under the licence chosen by the individual project.
