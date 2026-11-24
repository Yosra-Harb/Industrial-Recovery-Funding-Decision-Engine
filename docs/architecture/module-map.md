# Recommended Module Map v0.2

## Business Contexts

### 1. Identity & Access

Owns the global control plane for Organization, Tenant, actor identity references, tenant memberships, authorization assignments and access-scope evaluation.

Tenant is the primary data-isolation boundary.

Business contexts do not invent their own tenancy model.

Does not own external identity-provider credentials.

### 2. Factory Registry

Owns factory master/reference data and source/verification metadata.

### 3. Programme Catalogue & Policy

Owns FundingProvider, FundingProgramme, ProgrammePolicyVersion, PortfolioStrategy and Intervention catalogue configuration.

Published policy versions are immutable.

### 4. Application Intake & Validation

Owns RecoveryApplication, evidence metadata, submission lifecycle, validation and immutable submitted snapshots.

### 5. Evaluation Run

Orchestrates replaceable evaluation steps:

- eligibility
- diagnosis
- intervention matching
- indicator generation
- impact estimation

Owns EvaluationRun and exported step results.

### 6. Portfolio Optimization

Consumes a versioned candidate set, programme policy reference and budget snapshot.

Owns optimization execution and solver-neutral results.

Must not create funding commitments.

### 7. Decision & Governance

Owns RecommendationPackage, HumanDecision, overrides, approvals and BudgetCommitment.

This is the authoritative funding-governance boundary.

### 8. Monitoring & Outcomes

Owns post-funding observations and expected-vs-actual outcome records.

## Cross-Cutting / Edge

### Audit & Provenance

Consumes events from contexts and maintains tamper-evident provenance.

Business contexts must not import audit internals.

### Integration Gateway

Anti-corruption layer for host platforms and external partner systems.

Transforms external schemas into internal versioned contracts.

### Offline Sync

Edge capability for queued field operations, conflict detection and replay.

Does not grant independent offline funding authority.

### Model & Analytics — Future

Owns approved predictive model registry and analytics workflows.

It implements contracts such as ImpactEstimator without changing Evaluation or Optimization public contracts.

## Dependency Direction

Preferred flow:

```text
Identity & Access ──► authorization checks across API/application boundary

Factory Registry ───────────────┐
Programme Catalogue & Policy ──┼─► Application Intake
                               │
Application Intake ────────────┴─► Evaluation Run
Programme Policy ─────────────────► Evaluation Run

Evaluation Run ─► Candidate Contract ─► Portfolio Optimization
Programme Policy ─────────────────────► Portfolio Optimization
Decision & Governance ─► Budget Snapshot ─► Portfolio Optimization

Portfolio Optimization ─► Recommendation Contract ─► Decision & Governance

Decision & Governance ─► Monitoring & Outcomes
Monitoring & Outcomes ─► Future Model & Analytics

All contexts ─► safe events ─► Audit & Provenance
External systems ─► Integration Gateway ─► public application contracts
Offline client ─► Offline Sync ─► application contracts
```

## Forbidden Dependencies

- Optimization → OR-Tools types outside its infrastructure adapter
- Optimization → HumanDecision tables
- Evaluation → Optimization implementation internals
- any context → another context's private database tables
- business contexts → Audit storage internals
- UI → direct database
- offline client → authoritative approval/commitment without server authorization


## Multi-Tenant Boundary

All business contexts execute inside an explicit validated tenant context.

No business context may perform cross-tenant queries or joins.

A shared programme between institutions uses a dedicated joint tenant with explicit organization memberships.
