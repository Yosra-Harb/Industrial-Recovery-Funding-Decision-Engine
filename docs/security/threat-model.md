# Threat Model v0.1

## Purpose

This is an early architectural threat model for the Industrial Recovery Funding Decision Engine.

It is intentionally updated as implementation details become concrete.

## Primary Assets

- factory and enterprise data
- recovery applications
- evidence/documents
- programme rules and policy versions
- funding budgets and commitments
- evaluation results
- optimization inputs/results
- human decisions and overrides
- monitoring outcomes
- audit/provenance records
- identity/authorization data
- integration credentials and secrets

## Trust Boundaries

1. browser/mobile/PWA ↔ API
2. API ↔ identity provider
3. tenant ↔ tenant
4. bounded context ↔ bounded context
5. application ↔ PostgreSQL
6. application ↔ object storage
7. API ↔ external systems
8. web application ↔ background workers
9. offline device ↔ synchronization endpoint
10. evaluation/optimization runtime ↔ decision/governance boundary

## High-Risk Threats

### T-01 Cross-Tenant Data Leakage

Threat:
A user from Tenant A accesses Tenant B data through IDOR/BOLA, missing filters, background jobs, exports, cache keys or storage paths.

Controls:
- validated tenant context
- PostgreSQL RLS + FORCE RLS
- tenant-safe keys
- resource-scoped authorization
- tenant-aware cache/object-storage paths
- negative isolation tests

### T-02 Privilege Escalation

Threat:
A reviewer gains policy-publishing, override or commitment authority.

Controls:
- server-side RBAC + scope
- separation of duties
- explicit decision-authority grants
- privileged action audit
- short-lived tokens where supported

### T-03 Programme Rule Manipulation

Threat:
Eligibility, weights or constraints are changed to favor selected applicants.

Controls:
- immutable published policy versions
- author/approver separation
- content hashes
- audit trail
- controlled publication workflow

### T-04 Application / Evidence Tampering

Threat:
An applicant or insider modifies data/evidence after evaluation or substitutes documents.

Controls:
- immutable application snapshots
- evidence hashes
- versioning
- malware scanning
- object access control
- provenance metadata

### T-05 Optimization Manipulation

Threat:
Candidate set, budget snapshot, solver settings or objective configuration are altered without detection.

Controls:
- candidate-set snapshot/hash
- immutable budget snapshot input
- solver/runtime metadata
- optimizer adapter versioning
- deterministic/canonical input hashing
- optimization-run audit events

### T-06 Budget Oversubscription / Race Conditions

Threat:
Concurrent approvals commit more than programme budget.

Controls:
- authoritative Decision & Governance boundary
- transactional commitment operation
- row/advisory locking strategy
- concurrency tests
- optimizer cannot directly write commitments

### T-07 Audit Tampering

Threat:
An actor alters or deletes history to conceal a decision.

Controls:
- append-only model
- separate write privileges
- chained hashes or equivalent tamper evidence
- no UPDATE/DELETE for runtime role
- external backup/retention strategy

### T-08 Offline Device Compromise

Threat:
Lost/compromised device exposes cached data or queues unauthorized changes.

Controls:
- minimum offline data
- encrypted cache
- expiry/reauthentication
- token/session revocation
- server reauthorization on sync
- no offline final authority

### T-09 API Abuse / Enumeration

Threat:
Attackers enumerate IDs, scrape confidential records or overload expensive optimization endpoints.

Controls:
- tenant/resource authorization
- opaque UUIDs
- rate limits
- quotas for expensive operations
- request size/schema limits
- job queues for expensive workloads
- monitoring/alerts

### T-10 Malicious Uploads

Threat:
Uploaded evidence contains malware, dangerous file types or path tricks.

Controls:
- allow-listed MIME/types
- size limits
- malware scanning
- randomized object keys
- no direct execution
- access-controlled download
- content hashing

### T-11 Secret / Supply-Chain Compromise

Threat:
Leaked credentials or compromised dependency/container.

Controls:
- secret manager/environment injection
- no secrets in Git
- dependency pinning/lock files
- secret/SCA/container scanning
- SBOM
- key rotation

### T-12 Background Job Confused Deputy

Threat:
A worker processes a job without correct tenant/user/programme scope.

Controls:
- tenant_id and authorization context in job envelope
- worker revalidation
- idempotency keys
- no unscoped tenant jobs

### T-13 Insecure External Integration

Threat:
Partner APIs/webhooks impersonate systems or replay messages.

Controls:
- scoped machine identities
- OAuth2 client credentials and/or mTLS
- signed webhooks
- replay protection
- per-integration scopes
- anti-corruption layer validation

### T-14 Sensitive Data in Logs/Analytics

Threat:
PII or confidential factory data leaks through logs, traces or analytics.

Controls:
- safe structured logging
- redaction
- classification-aware telemetry
- correlation IDs instead of payload dumping

### T-15 Insider Misuse

Threat:
Authorized staff misuse legitimate access.

Controls:
- least privilege
- separation of duties
- access expiry
- audit/provenance
- export controls
- anomaly monitoring later
- break-glass process for exceptional administration

## Fail-Closed Rules

The system must fail closed when:

- tenant context is absent
- authorization cannot be verified
- policy version cannot be resolved
- decision-critical hash verification fails
- authoritative operation is attempted offline
- budget integrity cannot be guaranteed
- evidence/security validation required by policy fails

## Required Before Pilot

- concrete RLS policies and tests
- API authorization matrix
- upload-security implementation
- audit integrity implementation
- budget-concurrency test
- dependency/secret scans
- backup/restore test
- incident response procedure
- penetration/security assessment appropriate to deployment risk
