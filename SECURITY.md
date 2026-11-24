# Security and Confidentiality Policy

## Status

Architecture baseline v0.1.

## Security Objectives

The engine may process confidential enterprise, funding, operational, and decision data. Security is therefore a design constraint, not a post-development feature.

Mandatory principles:

- Security by Design
- Privacy by Design
- Least Privilege
- Deny by Default
- Data Minimization
- Defense in Depth
- Secure Defaults
- Full Auditability

## Authentication

Production deployments must use a standards-based identity provider compatible with OIDC/OAuth2.

The application must not implement custom password cryptography.

## Authorization

Authorization must be enforced server-side.

Initial model: RBAC, with future ABAC support where programme, organization, geography, or data classification requires finer controls.

Examples of roles:

- Factory Applicant
- Federation Reviewer
- Programme Manager
- Committee Reviewer
- Auditor
- System Administrator
- Read-Only Funder Observer

No role receives more permissions than required.

## Data Separation

Where feasible, identity/PII data must be separated from analytical and optimization data.

The optimization layer should operate on internal identifiers and decision-relevant attributes rather than unnecessary personal information.

## Encryption

Production requirements:

- TLS for data in transit
- Encryption at rest
- Encrypted backups
- Encrypted offline cache
- No secrets committed to Git
- Secret rotation supported

## Secrets

Secrets must be provided through secure environment or secret-management infrastructure.

Never commit:

- passwords
- API keys
- database credentials
- encryption keys
- access tokens
- production URLs containing credentials

`.env.example` must contain names/placeholders only.

## Audit Trail

Security-sensitive and decision-sensitive events must be auditable, including:

- authentication events where appropriate
- permission changes
- application state changes
- eligibility decisions
- optimization runs
- rule/configuration versions
- human overrides
- funding approvals/rejections
- data exports
- administrative changes

Audit records must be tamper-resistant and should not contain unnecessary sensitive payloads.

## Offline Security

Offline functionality must:

- cache only minimum required data
- encrypt local data
- expire cached data where appropriate
- queue writes safely
- prevent silent conflict overwrites
- re-authorize sensitive operations after reconnect when required
- treat the server as the authoritative source after synchronization

## Logging

Logs must never contain raw secrets or unnecessary sensitive enterprise data.

Structured logs should use identifiers, event types, correlation IDs, and safe metadata.

## Vulnerability Handling

Security defects are treated as priority defects.

Before production pilot:

- dependency scanning
- static analysis
- authentication/authorization tests
- API abuse tests
- backup/restore test
- threat-model review
- secret scanning

must be completed.

## Public Repository Rule

No real factory data, donor confidential material, production configuration, or security-sensitive implementation detail may be committed to a public repository.
