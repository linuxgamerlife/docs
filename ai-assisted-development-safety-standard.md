# AI-Assisted Development Safety Standard

Purpose

This standard defines how generative AI may be used in software development while maintaining security, reliability, and accountability. The goal is to ensure AI is used responsibly while maintaining strong automated quality and security controls.

AI may assist development, but trust must come from verified tooling and repeatable checks rather than blind acceptance of generated code.


Principles

AI output is untrusted code.

Code produced by AI must be treated the same as code from an unknown external contributor.

Automated verification is mandatory.

All AI-generated code must pass automated analysis and security tooling before it can be accepted.

Security-sensitive functionality must receive additional scrutiny.

Any code interacting with system privileges, external commands, or sensitive resources must undergo additional automated checks.


Approved Verification Methods

Projects using AI-assisted development must use reputable tooling to review and validate code.

Examples include:

Static analysis tools  
CodeQL analysis  
Compiler warnings treated as errors  
Sanitizer builds (ASan, UBSan, and optionally TSan)  
Dependency vulnerability scanning  
Automated testing frameworks

These tools must be integrated into the development workflow or CI pipeline where possible.


Security Issue Handling

All automated review tools must be monitored for issues.

High or Critical severity issues identified by analysis tools must be investigated and resolved before code is merged or released.

Medium severity issues should be reviewed and addressed where appropriate.

Low severity issues may be tracked for future improvement.


Usage Rules

AI may be used for:

Drafting boilerplate code  
Generating implementation examples  
Creating documentation or comments  
Refactoring code  
Explaining behaviour of existing code

AI must not be used for:

Automatically committing code without verification  
Bypassing automated analysis tools  
Ignoring High or Critical security findings


Testing Requirements

AI-generated code must pass all standard project checks including:

Successful compilation  
Static analysis  
Sanitizer builds where applicable  
Existing test suites

New functionality should include tests where practical.


Documentation

Where AI is used to assist development, it should be acknowledged in documentation or project transparency statements.

Code is accepted into the project based on the successful completion of automated verification checks and security analysis tooling defined by the project standards.

All software is distributed under the terms of the MIT licence and is provided "as is" without warranty.


Outcome

AI is used as a productivity tool while relying on trusted analysis tools and automated verification to maintain security, reliability, and code quality.
