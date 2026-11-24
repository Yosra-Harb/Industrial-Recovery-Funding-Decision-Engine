# ADR-002: Bounded Contexts Instead of Entity Modules

- Status: Accepted
- Date: 2026-11-24

## Decision

Use business bounded contexts rather than creating one module per entity.

Contexts:

1. Identity & Access
2. Factory Registry
3. Programme Catalogue & Policy
4. Application Intake & Validation
5. Evaluation Run
6. Portfolio Optimization
7. Decision & Governance
8. Monitoring & Outcomes

Audit/Provenance, Integration Gateway and Offline Sync are cross-cutting/edge capabilities.

## Reason

Entity-per-module design would create excessive coupling and temporal dependencies.

Bounded contexts align ownership, persistence and change boundaries.
