# Release Integrity Standard

Purpose

This standard defines how releases must be produced and verified to ensure users can trust distributed binaries.

Applications capable of performing system-level actions must provide verifiable releases.


Build Environment

Releases must be built in a controlled environment.

Manual local builds must not be used for official releases.

Continuous Integration (CI) must produce release artifacts.


Reproducibility

Build processes must be documented and automated.

The same source code must produce identical or functionally equivalent binaries when built under the same environment.


Release Signing

All official releases must be cryptographically signed.

Signing keys must be stored securely and access restricted.

Users must be able to verify the authenticity of a release before installing it.


Artifact Integrity

Each release must publish checksums for distributed files.

Checksums must be generated during the build process and verified before publication.


Dependency Control

Dependencies must be pinned to specific versions where possible.

External downloads during build must be avoided or verified using checksums or signatures.


Release Documentation

Each release must include:

- version number
- source commit reference
- build environment information
- changelog


Security Response

If a vulnerability is discovered:

- affected versions must be identified
- a patched release must be produced
- users must be notified through release notes or security advisories
