# ADR-009: Security as a Mandatory Architecture Gate

- Status: Accepted
- Date: 2026-11-24

## Context

The system processes confidential enterprise data and influences funding recommendations.

A security failure may expose data, alter eligibility, manipulate optimization, change programme rules, falsify evidence or corrupt the audit trail.

Security therefore cannot be treated as one infrastructure module or a pre-production checklist.

## Decision

Security is a cross-cutting architecture constraint applied to every bounded context, API, data store, background job, offline workflow and integration.

Every feature must define before merge:

1. authentication requirement
2. authorization and tenant/resource scope
3. data classification
4. encryption/storage requirements
5. audit/provenance events
6. input validation and abuse limits
7. secrets/dependency impact
8. offline/cache implications
9. failure mode and fail-closed behavior
10. required security tests

## Security Model

Adopt defense in depth with no implicit trust between:

- browser/client and API
- API and identity provider
- tenant and tenant
- bounded context and bounded context
- application and database
- application and object storage
- API and external partner
- web process and background worker
- offline device and authoritative server
- optimizer/estimator runtime and governance boundary

## Default Rules

- deny by default
- least privilege
- tenant isolation by default
- no security decision in frontend only
- no direct database access from UI
- no plaintext secrets in source/config repositories
- no unaudited privileged changes
- no authoritative offline funding decisions
- no mutable historical decision snapshots
- no silent fallback that weakens authorization or validation

## Sensitive Decision Integrity

Security controls protect not only confidentiality but decision integrity.

High-impact changes include:

- programme eligibility rules
- objective weights
- funding constraints
- risk/fairness parameters
- published intervention definitions
- optimization configuration
- human overrides
- budget commitments

These actions require strong authorization, provenance and configured separation of duties.

## Security Tests

Security acceptance includes, as applicable:

- authentication tests
- authorization-scope tests
- tenant-isolation tests
- IDOR/BOLA tests
- input/schema validation
- rate/size limit tests
- upload malware/path tests
- replay/idempotency tests
- offline conflict tests
- audit integrity tests
- secret/dependency scanning

## Consequence

A functionally correct feature is still incomplete if its security contract is undefined or untested.
