# Enterprise Architecture Policies

## Policy Status

**Baseline:** v0.2  
**Scope:** Entire Industrial Recovery Funding Decision Engine.

These policies are mandatory unless superseded by an approved ADR.

## P-01 — Bounded Context Independence

Each bounded context owns its domain logic, application services, persistence access, and internal implementation.

A change inside one context must not force unrelated contexts to change.

Prohibited by default:

- direct reads/writes to another context's private tables
- imports from another context's infrastructure internals
- circular dependencies
- shared mutable domain state
- duplicated policy logic across contexts

## P-02 — Extensibility

Major domain areas must support extension without core rewrites.

Examples include new programme types, rules, strategies, interventions, optimizers, monitoring indicators, country packs, and future predictors.

Prefer interfaces, strategies, registries, configuration and versioned contracts over large conditional blocks.

## P-03 — Modular Monolith First

The MVP uses a Modular Monolith with strict boundaries.

The architecture must preserve the option to extract a context into a service later without changing its public contract unnecessarily.

## P-04 — Contexts Are Not Entities

Entities do not automatically become modules.

Approved initial contexts:

1. Identity & Access
2. Factory Registry
3. Programme Catalogue & Policy
4. Application Intake & Validation
5. Evaluation Run
6. Portfolio Optimization
7. Decision & Governance
8. Monitoring & Outcomes

Cross-cutting / edge capabilities:

- Audit & Provenance
- Integration Gateway
- Offline Sync

Future:

- Model & Analytics

## P-05 — API First

The engine must not depend on one frontend.

External integrations use authenticated, versioned APIs or approved event contracts.

## P-06 — Configuration Over Hard Coding

Programme policy must be configuration-driven and versioned.

Do not hard-code programme-specific:

- eligible sectors
- funding limits
- document requirements
- portfolio weights
- risk thresholds
- fairness constraints
- objective priorities

Published policy versions are immutable. A change creates a new version.

## P-07 — Immutable Decision Snapshots

Every consequential evaluation or optimization run must reference immutable snapshots/hashes sufficient to reproduce the historical decision.

At minimum capture:

- application input snapshot
- programme policy version
- constraint version
- indicator definition version
- impact estimator version
- candidate-set hash/version
- optimizer adapter/version
- solver/runtime version
- solver parameters
- random seed when applicable
- concurrency/thread setting when relevant
- time limit/gap setting
- explanation version

## P-08 — Evaluation Run Orchestration

Eligibility, diagnosis, matching and impact assessment are pipeline steps coordinated by an Evaluation Run.

Pipeline steps must expose stable contracts and must not query downstream modules directly.

An Evaluation Run owns execution state, input snapshot reference, step results and version references.

## P-09 — Replaceable Algorithms

Workflow code must not depend directly on one solver or prediction library.

Required abstraction examples:

```text
ImpactEstimator
  ├── RuleBasedImpactEstimator
  └── FutureMLImpactEstimator

PortfolioOptimizer
  ├── ORToolsOptimizerAdapter
  └── FutureMILPOptimizerAdapter
```

Optimization contracts must contain domain-neutral problem/candidate data, not OR-Tools objects.

## P-10 — Budget Integrity

The Decision & Governance context owns approved allocations and budget commitments.

Optimization consumes a budget snapshot but does not directly mutate authoritative commitments.

Concurrent approvals must not oversubscribe a programme budget.

## P-11 — Security and Privacy by Design

Every context must define data classification, authorization scope and audit requirements.

Identity/PII should be separated from analytical data where feasible.

## P-12 — Authorization Scope From Day One

RBAC alone is insufficient.

Authorization must combine role with resource scope such as:

- organization
- programme
- factory ownership/assignment
- decision authority

Fine-grained ABAC may evolve later, but ownership/programme scoping is mandatory from the MVP.

## P-13 — Separation of Duties

High-impact policy changes and sensitive approvals must support four-eyes governance where configured.

At minimum, a user who authors a programme policy version must not be able to publish that same version alone when the deployment policy requires dual approval.

## P-14 — Offline Authority Boundary

Offline operation is for continuity, not independent funding authority.

MVP offline capabilities may include:

- draft application capture/update
- queued attachments/evidence metadata
- cached reference data
- non-authoritative client-side validation
- queued synchronization

Authoritative actions require reconnect and server authorization:

- programme policy publication
- eligibility finalization where it affects funding
- portfolio optimization as official run
- final approval/rejection
- override
- budget commitment

## P-15 — Offline Conflict and Idempotency

Offline writes must include client operation IDs/idempotency keys and base version references.

Synchronization must detect stale versions and must not silently overwrite server changes.

The server remains authoritative.

## P-16 — Audit & Provenance

Audit is cross-cutting and append-oriented.

Business contexts emit safe events; they do not depend on audit internals.

Decision-sensitive audit data must be tamper-evident using an approved append-only mechanism and separate write privileges.

Audit records should use pseudonymous/internal references where possible.

## P-17 — Data Retention and Privacy

Retention rules must be configurable by deployment/programme.

Audit immutability must not require permanent storage of unnecessary PII.

PII and business audit references should be separable so data can be deleted/anonymized according to policy without destroying decision provenance.

## P-18 — Persistence Ownership

Each context owns its persistence access.

Cross-context reporting uses read models, contracts, application services or events.

## P-19 — Contract Versioning

Breaking API or cross-context contract changes require explicit migration/compatibility strategy.

## P-20 — Testability and Architecture Enforcement

Required test classes:

- unit
- contract
- integration
- regression
- end-to-end
- security
- authorization-scope
- architecture boundary tests

CI must include an automated architecture/import boundary check.

## P-21 — Failure Isolation

Operations should be idempotent where relevant.

Retries, partial states and failures must be explicit.

Unsafe continuation must fail closed.

## P-22 — Observability

Production-capable modules expose structured logs, metrics and correlation IDs without leaking confidential data.

## P-23 — No Premature AI

AI/ML is added only when appropriate outcome data exists.

Rule-based estimation and mathematical optimization remain valid first-class implementations.

## P-24 — Public Repository Safety

A public repository must not contain real enterprise data, confidential donor/programme material, production secrets, credentials, internal security topology, or exploitable operational details.

## P-25 — Architecture Gate

A feature cannot be accepted if it:

- violates context ownership
- creates unnecessary tight coupling
- bypasses authorization scope
- bypasses audit/provenance
- hard-codes programme policy into core logic
- cannot be regression-tested
- breaks offline conflict guarantees
- makes historical decisions non-reproducible
- creates an irreversible dependency without an ADR
