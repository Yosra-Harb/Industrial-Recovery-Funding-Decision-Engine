# Enterprise Architecture Policies

## Policy Status

**Baseline:** v0.1  
**Scope:** Entire Industrial Recovery Funding Decision Engine.

These policies are mandatory unless superseded by an approved Architecture Decision Record (ADR).

---

## P-01 — Module Independence

Each module must own its domain logic and internal implementation.

A change inside one module must not force unrelated modules to change.

Modules communicate only through explicit contracts.

### Prohibited

- reading another module's private database tables directly
- importing another module's infrastructure layer
- circular dependencies
- sharing mutable internal state
- duplicating another module's business rules

---

## P-02 — Extensibility

Every major domain area must support future extension without rewriting the core.

Examples:

- new funding programmes
- new eligibility rules
- new portfolio strategies
- new intervention types
- new optimization algorithms
- new monitoring indicators
- future country/programme configurations

Prefer registries, strategies, interfaces, configuration, and versioned contracts over large conditional blocks.

---

## P-03 — Modular Monolith First

The MVP will use a **Modular Monolith with strict boundaries**.

Reason:

- lower operational complexity
- faster delivery
- easier local/offline development
- simpler testing
- preserves domain separation

Modules may later be extracted into independent services without changing their public contracts.

---

## P-04 — API First

The engine must not depend on one frontend.

External integrations use versioned APIs.

The existing host platform should be able to integrate without replacing its core system.

---

## P-05 — Configuration Over Hard Coding

Funding programme policies must be data/configuration driven.

Do not hard-code:

- eligible sectors
- funding limits
- portfolio weights
- priority objectives
- risk thresholds
- document requirements
- fairness constraints

Programme configuration must be versioned and auditable.

---

## P-06 — Security and Privacy by Design

Security/privacy review is mandatory for every module.

Sensitive data is minimized and access-controlled.

Identity data should be separated from analytical data where feasible.

---

## P-07 — Offline-Capable Design

Critical field workflows must tolerate unstable connectivity.

Offline operation uses:

```text
Encrypted local working cache
→ queued local operations
→ reconnect
→ authenticated synchronization
→ conflict detection
→ authoritative server reconciliation
```

Offline support must not weaken authorization or audit requirements.

---

## P-08 — Auditability

Every consequential decision must be reproducible.

Store references to:

- input snapshot/version
- programme configuration version
- rule version
- optimization algorithm/version
- model version when ML is used
- user/actor
- timestamp
- explanation
- override reason

---

## P-09 — Versioned Contracts

APIs and cross-module contracts are versioned when compatibility could break.

Breaking changes require explicit migration strategy.

---

## P-10 — Persistence Ownership

Each module owns its data access.

Cross-domain reporting should use approved read models, application services, or integration contracts instead of bypassing ownership boundaries.

---

## P-11 — Human-in-the-Loop

The engine recommends; authorized humans retain final funding authority.

Automation may process routine cases, but exceptions and final approvals remain governed by policy and authorization.

---

## P-12 — Replaceable Algorithms

Business workflows must not depend directly on one solver or ML library.

Examples:

```text
ImpactEstimator interface
  ├── RuleBasedImpactEstimator
  └── FutureMLImpactEstimator

PortfolioOptimizer interface
  ├── ORToolsOptimizer
  └── FutureMILPOptimizer
```

---

## P-13 — Testability

Architecture must support:

- unit tests
- contract tests
- integration tests
- end-to-end tests
- regression tests
- security tests

A module that cannot be tested independently is considered too tightly coupled.

---

## P-14 — Observability

Production-capable modules must provide structured logs, metrics, trace/correlation identifiers, and actionable errors without leaking confidential data.

---

## P-15 — Failure Isolation

A non-critical module failure should not corrupt the entire workflow.

Where possible:

- operations are idempotent
- retryable actions are explicit
- failures are recorded
- partial state is identifiable
- unsafe continuation fails closed

---

## P-16 — Data Portability

The system must support controlled import/export through documented formats and APIs to avoid vendor lock-in.

---

## P-17 — No Premature AI

AI/ML is added only when justified by sufficient and appropriate outcome data.

Rules and optimization remain valid first-class implementations.

---

## P-18 — Architecture Gate

A feature cannot be accepted if it:

- violates module ownership
- creates unnecessary tight coupling
- bypasses authorization
- bypasses audit logging
- hard-codes programme-specific policy into core logic
- cannot be regression-tested
- creates an irreversible dependency without an ADR
