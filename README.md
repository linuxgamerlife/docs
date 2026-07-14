# Development Standards

## Important Notice

I am not a professional software developer. The projects I create rely heavily on generative AI to assist with code development and production.

Applications are tested extensively and reviewed using automated analysis and security tooling where possible. Security testing and verification are performed continuously as projects evolve.

Despite these precautions, software may still contain bugs, vulnerabilities, or unintended behaviour.

Any use of software from these repositories is done entirely at the user's own risk.

All software is provided under the terms of the MIT licence and is distributed **"as is" without warranty of any kind.**


## Purpose

This repository contains the development and security standards used across LinuxGamerLife projects.

These standards provide guardrails for building software in a responsible and structured way, particularly when AI-assisted development is involved.

They focus on:

- engineering quality
- security architecture
- safe handling of privileged operations
- release integrity
- responsible AI-assisted development


## Standards

### C++ / Qt Development Standard

Defines the engineering rules for C++ and Qt applications.

Focus areas include:

- modern C++ practices
- memory safety
- resource management
- Qt best practices
- static analysis
- sanitizers
- defensive coding

File  
`cpp-qt-development-standard.md`


### Privileged Operations Standard

Defines how applications that perform system-level operations must be designed.

This standard applies to tools that interact with:

- package managers
- system configuration
- repositories
- hardware or drivers
- administrative system tasks

Focus areas include:

- privilege separation
- minimal root code
- allow-listed operations
- input validation
- safe failure behaviour

File  
`privileged-operations-standard.md`


### Secure Design Review Standard

Defines the process for evaluating security risks during the design phase before development begins.

The goal is to identify architectural risks early rather than fixing them after implementation.

Focus areas include:

- identifying trust boundaries
- analysing affected system assets
- evaluating misuse scenarios
- applying least privilege principles

File  
`secure-design-review-standard.md`


### Release Integrity Standard

Defines how releases must be produced and verified so users can trust distributed binaries.

Focus areas include:

- controlled build environments
- CI-generated release artifacts
- release signing
- checksums for binaries
- dependency control
- vulnerability response procedures

File  
`release-integrity-standard.md`


### AI-Assisted Development Safety Standard

Defines how generative AI may be used responsibly during development.

AI is treated as a productivity tool rather than a trusted source of production-ready code.

Code generated with AI must pass automated verification using reputable tooling such as:

- static analysis
- CodeQL
- compiler warnings
- sanitizers
- automated testing

High or critical issues identified by these tools must be investigated and addressed before release.

File  
`ai-assisted-development-safety-standard.md`


### Secure Web Application Development Standard

Defines the security rules for web applications, APIs, browser-based interfaces, and embedded remote web content.

Focus areas include:

- authentication and authorisation
- input validation and injection prevention
- browser security controls
- API and session security
- dependency and supply chain security
- secure handling of embedded remote content

File

`secure-web-application-development-standard.md`


## How These Standards Work Together

These standards form a layered development model.

1. Features are reviewed using secure design principles.
2. Code must follow strict C++ and Qt engineering standards.
3. Privileged operations are isolated and tightly controlled.
4. Automated tooling verifies security and reliability.
5. Releases are built and distributed with verifiable integrity.

This approach helps ensure LinuxGamerLife software remains transparent, testable, and as safe as reasonably possible.


## Scope

These standards apply to:

- software developed under the LinuxGamerLife project
- applications that interact with system configuration
- projects using AI-assisted development

| Standard | Status |
|--------|--------|
| C++ / Qt Development Standard | Active |
| Privileged Operations Standard | WIP |
| Secure Design Review Standard | WIP |
| Release Integrity Standard | WIP |
| AI-Assisted Development Safety Standard | WIP |
| Secure Web Application Development Standard | Active |


## Licence

These standards are published under the MIT licence.

Software developed using these standards may be distributed under the licence chosen by the individual project.
