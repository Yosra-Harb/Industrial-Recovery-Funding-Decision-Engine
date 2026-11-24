# Industrial Recovery Funding Decision Engine

An institutional, API-first decision engine for automating and optimizing industrial recovery funding workflows.

## Purpose

The engine is designed to integrate with existing federation, government, donor, or development-programme platforms without requiring replacement of their current systems.

It transforms:

```text
Factory Data
+ Recovery Application
+ Funding Provider
+ Funding Programme
+ Portfolio Strategy
+ Programme Constraints
        ↓
Validation
        ↓
Eligibility
        ↓
Recovery Needs Diagnosis
        ↓
Intervention Matching
        ↓
Impact & Priority Assessment
        ↓
Portfolio Optimization
        ↓
Explainable Recommendation
        ↓
Human Review / Approval
        ↓
Post-Funding Monitoring
        ↓
Outcome Learning
```

## Core Engineering Principles

1. **Module independence is mandatory.**
2. **Every module must be extensible and replaceable.**
3. **API-first and headless integration.**
4. **Security and privacy by design.**
5. **Offline-capable workflows for unstable connectivity.**
6. **Human-in-the-loop for final funding decisions.**
7. **Full auditability and versioning.**
8. **Configuration over hard-coded programme rules.**
9. **Automated tests and regression protection.**
10. **Scalable architecture without premature microservices.**

## Initial Architecture

The MVP starts as a **Modular Monolith with strict domain boundaries**.

Planned modules:

- Factory
- Funding Provider
- Funding Programme
- Portfolio Strategy
- Recovery Application
- Validation
- Eligibility
- Recovery Diagnosis
- Intervention
- Assessment
- Optimization
- Decision
- Monitoring
- Audit
- Integration

Modules communicate through explicit contracts. A module must not depend on another module's internal implementation details or directly access another module's private persistence layer.

## Decision Model

The engine does not only rank factories. It selects the best **Factory–Intervention portfolio** under budget and programme constraints.

Simplified objective:

```text
maximize Total Expected Recovery Impact

subject to:
  Total Funding <= Available Budget
  Eligibility Rules
  Maximum Funding per Factory
  Sector Constraints
  Risk Limits
  Fairness Constraints
  Employment Targets
  Programme-Specific Constraints
```

## MVP Scope

The first MVP includes:

- Factory master data
- Funding provider and programme configuration
- Portfolio strategy
- Recovery applications
- Validation
- Eligibility
- Recovery-needs diagnosis
- Intervention matching
- Priority and impact assessment
- Portfolio optimization
- Scenario analysis
- Explainable recommendations
- Human review / override
- Basic audit trail
- Prototype post-funding monitoring

## Deferred Capabilities

These are intentionally deferred until justified by data and operational needs:

- Production-grade predictive ML
- Causal inference
- Multi-country deployment
- Supply-chain graph optimization
- Autonomous funding decisions
- Payment/banking processing
- Full microservices deployment

## Technology Direction

- Backend: Python + FastAPI
- Database: PostgreSQL
- Optimization: Google OR-Tools initially
- Frontend: React / PWA-capable
- Deployment: Docker
- Auth: OIDC / OAuth2 compatible
- Testing: Unit + Contract + Integration + End-to-End + Regression

Technology choices may evolve, but module contracts and architectural boundaries must remain stable.

## Repository Status

**Phase:** Architecture baseline and MVP definition.

See:

- `docs/architecture/enterprise-policies.md`
- `docs/product/scope.md`
- `docs/product/workflow.md`
- `docs/data/data-model.md`
- `SECURITY.md`

## Non-Negotiable Rule

> A change inside one module must not require unrelated modules to be rewritten.

Any exception requires an explicit Architecture Decision Record (ADR).
