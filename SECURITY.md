# Security and Confidentiality Policy v0.3

## Security Objectives

The engine may process confidential enterprise, funding, evidence and decision data.

Mandatory principles:

- Security by Design
- Privacy by Design
- Least Privilege
- Deny by Default
- Data Minimization
- Defense in Depth
- Secure Defaults
- Full Auditability
- Separation of Duties

## Authentication

Production deployments use a standards-based identity provider compatible with OIDC/OAuth2.

Do not implement custom password cryptography.

Service-to-service integration must use scoped machine identities, e.g. OAuth2 client credentials and/or mTLS according to deployment requirements.

## Multi-Tenant Security Boundary

The system is multi-tenant.

A tenant is the primary data-isolation boundary.

Tenant business data is isolated by:

- server-validated tenant membership
- resource-scoped authorization
- tenant_id on tenant-owned records
- PostgreSQL RLS for production-capable deployments
- tenant-aware cache/queue/object-storage/export keys
- negative tenant-isolation tests

The client cannot authorize its own tenant context.

Routine administrators do not automatically receive access to tenant business data.

## Authorization

Authorization is enforced server-side.

The MVP uses:

**RBAC + mandatory resource scoping**

Scopes may include:

- organization
- programme
- factory ownership/assignment
- review queue
- decision authority

RBAC alone is not sufficient.

Fine-grained ABAC may be added later if justified.

## Separation of Duties

Sensitive policy and decision operations must support four-eyes control where configured.

Examples:

- programme policy author cannot self-publish the same version
- rule changes require approval
- privileged system administration does not automatically grant business decision visibility
- override authority is separate from ordinary review where policy requires it

Conflict-of-interest declaration/checking must be supported for reviewers where required.

## Data Separation and Classification

Identity/PII should be separated from analytical data where feasible.

Removing a name does not automatically make data non-sensitive; sector, geography and size may still re-identify a factory.

Access control and export policy must follow data classification.

## Encryption

Production requirements:

- TLS in transit
- encryption at rest
- encrypted backups
- encrypted offline cache
- secret rotation
- no secrets in Git

## Secrets

Never commit passwords, API keys, database credentials, encryption keys, access tokens or production credential-bearing URLs.

## Programme and Rule Integrity

Programme policy versions must be immutable once published.

Every policy version requires content hashing, author/approver provenance and publication timestamp.

Unauthorized rule or weight changes are treated as security-sensitive events.

## Decision Reproducibility

Official optimization runs must record enough execution metadata to reproduce or explain historical results, including solver/runtime versions and parameters.

Non-deterministic settings must be explicit.

## Audit Trail

Decision/security-sensitive events must be recorded.

Audit design requirements:

- append-oriented
- tamper-evident
- separate write privileges from ordinary business tables
- pseudonymous/internal actor/entity references where possible
- chained/event hashes or equivalent integrity control
- retention policy separate from PII retention

## Privacy and Retention

Auditability does not justify indefinite storage of unnecessary PII.

Deployments must define retention periods.

Identity data and audit/provenance references should be separable to allow deletion/anonymization where policy/law requires without destroying decision provenance.

## Offline Security

Offline MVP supports only limited non-authoritative workflows.

Allowed:

- draft data capture/edit
- minimum cached reference data
- queued evidence metadata/uploads
- local advisory validation
- queued synchronization

Authoritative operations require online server authorization:

- programme policy publication
- official evaluation finalization
- official portfolio optimization
- approval/rejection
- override
- budget commitment

Offline writes must carry idempotency keys and base-version references.

No silent conflict overwrite is allowed.

## Device Risk

Offline cache is assumed vulnerable if the endpoint itself is compromised.

Therefore:

- minimize cached data
- use platform-supported non-exportable key material where feasible
- expire/clear sensitive cache
- require re-authentication after defined conditions
- support administrative session/token revocation

Remote device wipe is deployment-dependent and not assumed for the MVP.

## API and Partner Security

Integrations must support:

- scoped service identities
- rate limiting
- idempotency keys
- replay protection where applicable
- signed webhooks when webhooks are used
- request size limits
- schema validation
- authorization on every resource operation

## Evidence / Upload Security

Document uploads require:

- allow-listed content types
- size limits
- malware scanning before trusted use
- randomized storage identifiers
- no direct execution
- classification metadata
- integrity hash
- access-controlled retrieval

## Logging

Logs must not contain raw secrets or unnecessary confidential data.

Use structured event types, IDs, correlation IDs and safe metadata.

## Supply Chain

Before production pilot:

- dependency scanning
- secret scanning
- static analysis
- locked dependency versions
- SBOM generation
- container/image scanning where applicable

## Backup and Recovery

Before production pilot define and test:

- RPO
- RTO
- encrypted backup process
- restore procedure
- recovery authorization

## Incident Response

Before production pilot define incident ownership, containment, credential revocation, evidence preservation and stakeholder notification process.

## Public Repository Rule

No real factory data, confidential donor/programme material, production configuration, credentials, internal deployment secrets, or exploitable security topology may be committed to a public repository.

## Vulnerability Reporting

A public security contact/process should be published before external/public deployment.


## Security Architecture Gate

Every feature requires a security contract before completion:

- authentication
- tenant/resource authorization
- data classification
- encryption/storage
- audit events
- input validation/abuse limits
- offline/cache behavior
- failure mode
- security tests

A feature that is functionally correct but lacks this definition is incomplete.

## Fail-Closed Requirements

Authoritative operations are blocked when:

- tenant context is absent or invalid
- authorization cannot be confirmed
- required policy/snapshot integrity checks fail
- official funding action is attempted offline
- budget integrity cannot be guaranteed

## Row-Level Security

Production-capable PostgreSQL deployments use RLS for tenant-owned tables.

Runtime database roles must not be superusers and must not have BYPASSRLS.

Tenant context must be transaction-local when connection pooling is used.

RLS supplements, but does not replace, application authorization.
