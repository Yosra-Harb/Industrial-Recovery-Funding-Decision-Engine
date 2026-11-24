# Industrial Recovery Funding Decision Engine

An institutional, API-first engine for automating and optimizing industrial recovery funding workflows.

## Purpose

The engine integrates with existing federation, government, donor, or development-programme platforms without requiring replacement of their current systems.

It transforms:

```text
Factory Data
+ Recovery Application
+ Funding Provider / Programme Policy
+ Portfolio Strategy
+ Programme Constraints
        ↓
Application Intake & Validation
        ↓
Evaluation Run
  ├─ Eligibility
  ├─ Recovery Diagnosis
  ├─ Intervention Matching
  └─ Impact / Priority Estimation
        ↓
Candidate Factory–Intervention Set
        ↓
Portfolio Optimization
        ↓
Recommendation Package
        ↓
Human Review / Approval
        ↓
Funding Decision + Budget Commitment
        ↓
Monitoring & Outcomes
        ↓
Expected vs Actual Analysis
        ↓
Future Learning / Re-optimization
```

## Non-Negotiable Engineering Principles

1. Module independence.
2. Extensibility and replaceable implementations.
3. Explicit, versioned contracts.
4. API-first and headless integration.
5. Security and privacy by design.
6. Offline-capable field workflows with strict authority boundaries.
7. Human-in-the-loop for final funding authority.
8. Full auditability and reproducibility.
9. Configuration over hard-coded programme policy.
10. Automated contract, integration, regression and security tests.
11. Scalable architecture without premature microservices.

## Bounded Contexts

The MVP starts as a **Modular Monolith with strict domain boundaries**.

Primary contexts:

- Identity & Access
- Factory Registry
- Programme Catalogue & Policy
- Application Intake & Validation
- Evaluation Run
- Portfolio Optimization
- Decision & Governance
- Monitoring & Outcomes

Cross-cutting / edge capabilities:

- Audit & Provenance
- Integration Gateway
- Offline Sync

Future:

- Model & Analytics

Entities are not automatically modules. Cross-context communication occurs through published contracts, application services, immutable snapshots, or approved events.

## Decision Model

The engine does not only rank factories. It selects the best **Factory–Intervention portfolio** under budget and programme constraints.

Simplified objective:

```text
maximize Total Expected Recovery Impact

subject to:
  Total Funding + Existing Commitments <= Available Budget
  Eligibility Rules
  Maximum Funding per Factory
  Sector Constraints
  Risk Limits
  Fairness Constraints
  Employment Targets
  Programme-Specific Constraints
```

The optimization boundary is solver-neutral. Core contracts must not expose OR-Tools types.

## MVP Scope

The first MVP includes:

- Factory reference/master data
- Programme policy configuration and versioning
- Recovery applications and evidence metadata
- Validation
- Eligibility
- Recovery-needs diagnosis
- Intervention matching
- Rule-based impact estimation
- Candidate generation
- Portfolio optimization
- Scenario analysis
- Explainable recommendation package
- Human review / override
- Budget commitment record
- Audit/provenance
- Prototype post-funding monitoring

## Deferred Capabilities

- Production-grade predictive ML
- Causal inference
- Multi-country deployment
- Supply-chain graph optimization
- Autonomous final funding decisions
- Payment/banking execution
- Full microservices deployment

## Technology Direction

- Backend: Python + FastAPI
- Database: PostgreSQL
- Optimization: Google OR-Tools initially
- Frontend: React / PWA-capable
- Deployment: Docker
- Auth: OIDC / OAuth2 compatible
- Testing: Unit + Contract + Integration + End-to-End + Regression + Security

Technology choices may evolve, but public contracts and module boundaries must remain stable.

## Architecture References

- `docs/architecture/enterprise-policies.md`
- `docs/architecture/module-map.md`
- `docs/architecture/contracts.md`
- `docs/product/scope.md`
- `docs/product/workflow.md`
- `docs/data/data-model.md`
- `SECURITY.md`

## Non-Negotiable Rule

> A change inside one context must not require unrelated contexts to be rewritten.

Any exception requires an explicit Architecture Decision Record (ADR).


## Multi-Tenant Security

The same system is designed to support multiple institutions.

A Tenant is the primary data-isolation workspace.

Tenant business data is isolated using server-side authorization, tenant-aware data models, PostgreSQL Row-Level Security in production-capable deployments, and tenant-scoped cache/queue/object-storage/export rules.

Cross-tenant access is denied by default.
