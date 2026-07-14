# Secure Web Application Development Standard

## Purpose

This standard defines the minimum security requirements for web applications, browser-based interfaces, APIs, and embedded remote web content.

The goal is to build applications that are secure by design, secure by default, and resistant to common web attacks.

All web application code must follow this standard unless an exception is reviewed, approved, documented, and time limited.


## Security Baseline

Applications must use the current stable OWASP Application Security Verification Standard (ASVS) as their verification baseline.

The default target is **OWASP ASVS Level 2**.

| Application Risk | Minimum Target |
|---|---|
| Low-risk application with no sensitive data | ASVS Level 1, with documented approval |
| Most applications and applications handling sensitive data | ASVS Level 2 |
| High-value, safety-critical, or highly sensitive application | ASVS Level 3 |

Compliance with a baseline does not replace application-specific threat modelling.


## Secure Design

Significant features must undergo a secure design review before implementation.

The review must identify:

- assets and sensitive data
- users, roles, and privilege levels
- entry points
- trust boundaries
- external services
- abuse cases
- required security controls
- residual risks

Applications must follow these principles:

| Principle | Requirement |
|---|---|
| Least Privilege | Components receive only the permissions they require |
| Deny by Default | Access is denied unless explicitly permitted |
| Defence in Depth | No single control is relied on for complete protection |
| Data Minimisation | Only required data is collected and retained |
| Secure Failure | Security-control failure stops the protected operation safely |
| Server Authority | Security decisions are enforced by trusted server or native code |

Client-side code must always be treated as untrusted.


## Authentication

Authentication must use maintained libraries and established protocols.

Custom authentication protocols or cryptographic schemes must not be invented.

Requirements:

- default, shared, or hard-coded credentials are prohibited
- multi-factor authentication must be available for privileged accounts
- administrative access should require multi-factor authentication
- login, recovery, and enrolment operations must be rate limited
- authentication responses should avoid unnecessary account enumeration
- recovery and verification tokens must be random, single use, and short lived
- re-authentication must protect sensitive account changes where appropriate
- delegated identity tokens must have their signature, issuer, audience, expiry, and required claims validated

Passwords must:

- be hashed with a modern password-specific algorithm
- use a unique salt
- use an appropriate work factor
- never be logged
- never be stored reversibly
- never be emailed or returned by an API


## Session Management

Session identifiers must be cryptographically random and must not contain sensitive or predictable information.

Session cookies must use:

- `Secure`
- `HttpOnly`
- an appropriate `SameSite` value

Host-only session cookies should use the `__Host-` prefix, `Path=/`, and no `Domain` attribute.

Additional requirements:

- session identifiers must rotate after login and privilege changes
- sessions must have risk-appropriate idle and absolute expiry
- logout must invalidate the server-side session or refresh credential
- password reset and suspected compromise should revoke existing sessions
- sensitive responses must use appropriate cache controls

Authentication tokens should not be stored in `localStorage` or `sessionStorage`. Any exception requires a documented threat assessment.


## Authorisation

Every request for protected data or functionality must be authorised by trusted code.

Requirements:

- access must be denied by default
- access checks must use the authenticated identity, requested resource, operation, and ownership or tenant context
- object identifiers must never be treated as proof of access
- multi-tenant data isolation must be enforced on every applicable data path
- administrative and bulk operations must have separate access rules
- role, permission, and ownership changes must be logged
- user-interface visibility must not be used as an access control


## Input Validation

All external input must be treated as untrusted.

This includes:

- form and URL values
- request headers and cookies
- uploaded files
- API responses
- database content
- message queues
- IPC messages
- environment variables

Input must be validated against the expected:

- type
- length
- range
- format
- structure
- permitted values

Server-side validation is required even when client-side validation exists.

Malformed input must be rejected early and safely.


## Injection Prevention

Database operations must use parameterised queries or a correctly configured query builder.

User input must not be concatenated into:

- SQL or NoSQL queries
- operating-system commands
- templates
- code
- expressions
- dynamic imports

Shell execution should be avoided. When unavoidable, programs must be executed directly with explicit, validated argument lists.

Untrusted data must not control deserialised object types.

Redirect destinations must be restricted to approved locations.

Server-side URL requests must protect against server-side request forgery by validating schemes, hosts, addresses, redirects, and permitted networks.


## Cross-Site Scripting Prevention

Framework auto-escaping must remain enabled.

Untrusted output must be encoded for its exact context, including:

- HTML
- HTML attributes
- URLs
- CSS
- JavaScript

Requirements:

- untrusted strings must not be passed to `innerHTML` or equivalent HTML interpretation APIs
- rich HTML must use a maintained allow-list sanitiser
- `eval`, `new Function`, and string-based timers must not process untrusted data
- Trusted Types should restrict dangerous DOM sinks where supported and practical
- Content Security Policy must be used as defence in depth, not as a replacement for encoding or sanitisation


## Cross-Site Request Forgery Prevention

State-changing requests authenticated with cookies must have cross-site request forgery protection.

Appropriate controls include:

- framework-provided anti-CSRF tokens
- same-site cookies
- origin validation
- Fetch Metadata validation
- required custom request headers for same-origin API clients

State changes must not use `GET`, `HEAD`, or other safe HTTP methods.


## Browser Security Headers

Production applications must define and test security headers appropriate to their architecture.

The starting baseline is:

```http
Content-Security-Policy: default-src 'self'; object-src 'none'; base-uri 'none'; frame-ancestors 'none'; form-action 'self'
X-Content-Type-Options: nosniff
Referrer-Policy: strict-origin-when-cross-origin
Permissions-Policy: camera=(), microphone=(), geolocation=()
Cross-Origin-Opener-Policy: same-origin
```

Requirements:

- Content Security Policy must be extended only for required resources
- script nonces or hashes should be preferred
- `unsafe-eval` must not be enabled
- `unsafe-inline` should be avoided
- major CSP changes should be tested with report-only mode before enforcement
- framing must be denied unless explicitly required
- permitted framing origins must be narrowly listed
- HTTPS-only applications must use HTTP Strict Transport Security after all affected hosts are verified
- unneeded browser capabilities must be disabled with `Permissions-Policy`
- `X-XSS-Protection` should be omitted or set to `0`
- cross-origin isolation headers must be tested for compatibility
- third-party scripts should be self-hosted
- fixed external scripts and styles should use Subresource Integrity when supported

The example policy is a starting point and must not be copied without application-specific testing.


## Cross-Origin Resource Sharing

CORS must remain disabled unless cross-origin browser access is required.

When required:

- allowed origins must be explicit complete origins
- reflected origins must be validated against an allow list
- permitted methods and headers must be limited
- wildcard origins must not be combined with credentials
- CORS must not be treated as authentication or authorisation


## API Security

APIs must:

- authenticate and authorise every protected operation
- validate request and response schemas
- reject unsupported content types
- limit body, upload, pagination, and batch sizes
- rate limit expensive or abuse-prone operations
- return safe errors without stack traces, secrets, queries, or internal paths
- use an explicit version and deprecation policy
- prevent abandoned endpoints from remaining exposed

Webhooks must validate a cryptographic signature over the exact received content, reject stale requests, and handle replay safely.

GraphQL services must enforce authorisation at the data or resolver layer and limit query depth, complexity, aliases, and batching.


## File Upload and Download Safety

File handling must enforce limits for:

- file size
- file count
- filename length
- permitted file types

Requirements:

- file validation must not rely only on extensions or client-provided content types
- generated storage names must be used
- user filenames must not determine filesystem paths
- uploads must be stored outside executable application paths unless safely isolated
- archive extraction must prevent path traversal and decompression bombs
- downloads must use correct content types, `nosniff`, and safe content disposition
- malware scanning should be used when justified by file type and risk


## Secrets, Cryptography, and Transport

Production traffic containing authentication, personal, sensitive, or privileged data must use TLS.

Certificate verification must never be disabled in production.

Secrets must not be:

- committed to source control
- placed in client-side code
- written to logs
- included in URLs
- stored in example configuration

Secrets must use an approved secret-management mechanism, be scoped to the minimum required service and environment, and support rotation.

Only standard, maintained cryptographic libraries may be used.

Security-sensitive random values must use a cryptographically secure random source.


## Data Protection and Privacy

Applications must collect and retain only data required for a documented purpose.

Requirements:

- sensitive data must not appear in URLs, referrers, analytics properties, or unnecessary browser storage
- retention and deletion rules must be implemented
- data exports and account deletion must be authorised
- telemetry must be documented and proportionate
- consent and preference decisions must be respected where required
- backups and replicas must receive equivalent protection and retention controls


## Logging and Error Handling

Applications must log security-relevant events where applicable, including:

- authentication success and failure
- account recovery
- security-setting changes
- failed authorisation
- validation bypass attempts
- privileged operations
- security-control configuration changes

Logs should contain timestamps, event types, outcomes, and correlation identifiers.

Logs must never contain:

- passwords
- session identifiers
- access or refresh tokens
- cryptographic keys
- full payment details
- unnecessary personal content

Log data must be encoded to prevent log injection. Access and retention must be restricted and documented.

User-facing errors must be useful without exposing stack traces, queries, internal paths, secrets, or implementation details.


## Dependency and Supply Chain Security

Dependencies must:

- come from trusted sources
- be pinned using a committed lockfile or equivalent mechanism
- be scanned automatically for vulnerabilities
- be updated within a period appropriate to severity and exposure
- be removed when unused

Material new dependencies and package installation scripts must receive review.

Production builds must be automated and reproducible where practical.

Build and release credentials must be isolated and use least privilege.

Release artefacts should include provenance and a software bill of materials where appropriate.

Third-party scripts and services must be inventoried and reviewed periodically.


## Embedded Remote Web Content

Desktop or mobile applications embedding remote web content must treat that content as untrusted.

Requirements:

- remote content must not have Node.js, shell, filesystem, process, or unrestricted native access
- remote and trusted local content must use separate origins or renderer contexts
- native capabilities must use narrow, typed, validated operations
- message sender, frame, and origin must be verified
- navigation, new windows, downloads, and permission requests must be intercepted and allow listed
- hardware access must validate both the requesting origin and selected device
- remote content must not broaden its own permissions
- remote site data must use an application-specific profile and be resettable
- remote-site compromise must not grant native code execution


## Development Verification

Security requirements and abuse cases must be part of acceptance criteria.

Projects must use automated verification appropriate to their technology, including:

- type checking
- linting
- secret scanning
- static analysis
- dependency vulnerability scanning
- automated tests

Tests must cover relevant:

- authentication
- authorisation
- validation
- tenant isolation
- security boundaries
- failure paths

Security-sensitive code must receive peer review.

Dynamic security testing should run against a production-like environment.

High-risk parsers and protocol handlers should receive fuzz testing.

Security fixes should include regression tests.


## Deployment and Operations

Production systems must:

- run with least privilege
- disable debug modes and development endpoints
- prevent public access to private databases, caches, queues, and management ports
- restrict administrative interfaces and require strong authentication
- use controlled configuration and deployment automation
- define severity-based security patch targets
- monitor service health and relevant security signals
- maintain tested backup and restoration procedures
- maintain incident response and credential rotation procedures


## Release Requirements

A production release must not proceed until:

- applicable mandatory controls are implemented
- automated security checks pass
- unresolved findings have approved, time-limited exceptions
- protected operations have authorisation tests
- new input paths have validation and abuse tests
- dependency and secret scans contain no unreviewed critical findings
- production configuration has been checked for unsafe settings
- material threat-model changes have been reviewed

High-risk applications should receive independent penetration testing before initial release and after major attack-surface changes.


## Security Exception Process

An exception must document:

- the requirement being waived
- affected components and data
- threat and impact analysis
- reason compliance is not currently practical
- compensating controls
- accountable owner and approver
- expiry date
- tracked remediation work

Expired exceptions must be resolved or renewed before release.


## Compliance Checklist

Before merging a material web application change:

- [ ] secure design and trust boundaries reviewed
- [ ] sensitive data and abuse cases identified
- [ ] authentication enforced by trusted code
- [ ] authorisation enforced for each protected resource and action
- [ ] external input validated against expected values
- [ ] output encoded for the correct context
- [ ] database operations parameterised
- [ ] cookie-authenticated state changes protected against CSRF
- [ ] CSP, CORS, browser permissions, and navigation use least privilege
- [ ] no secrets or sensitive data added to code, logs, URLs, or client storage
- [ ] dependencies and third-party content reviewed
- [ ] security-sensitive behaviour covered by tests
- [ ] failures are safe and error messages do not expose internals
- [ ] security documentation and threat models updated
- [ ] no unapproved high or critical security findings remain


## References

- [OWASP Application Security Verification Standard](https://owasp.org/www-project-application-security-verification-standard/)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
- [OWASP Content Security Policy Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Content_Security_Policy_Cheat_Sheet.html)
- [OWASP HTTP Security Response Headers Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/HTTP_Headers_Cheat_Sheet.html)
- [MDN Web Security](https://developer.mozilla.org/en-US/docs/Web/Security)
- [MDN Secure Cookie Configuration](https://developer.mozilla.org/en-US/docs/Web/Security/Practical_implementation_guides/Cookies)
- [MDN Subresource Integrity](https://developer.mozilla.org/en-US/docs/Web/Security/Defenses/Subresource_Integrity)
- [CISA Secure by Design](https://www.cisa.gov/securebydesign)
- [NIST Secure Software Development Framework](https://csrc.nist.gov/Projects/ssdf)
