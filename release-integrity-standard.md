# Release Integrity Standard

## Purpose

This standard defines how releases must be produced and verified so users can trust distributed binaries.

Applications capable of performing system-level actions must provide verifiable releases.


## Build Environment

Official releases must be built in a controlled environment.

Requirements:

- builds must occur in CI where possible
- manual local builds must not be used for official releases


## Reproducible Builds

Build processes must be documented and automated.

The same source code should produce equivalent binaries when built in the same environment.


## Release Signing

All official releases must be cryptographically signed.

Signing keys must be stored securely and access restricted.

Users must be able to verify the authenticity of a release.


## Artifact Integrity

Each release must publish checksums for distributed files.

Checksums must be generated during the build process and verified before publication.


## Dependency Control

Dependencies should be pinned to specific versions where possible.

External downloads during build must be avoided or verified.


## Security Response

If a vulnerability is discovered:

- affected versions must be identified
- patched releases must be produced
- users must be notified through release notes or advisories


## Compliance Checklist

Before publishing a release:

- [ ] build produced in CI
- [ ] release artifacts signed
- [ ] checksums published
- [ ] dependencies verified
- [ ] release notes prepared
