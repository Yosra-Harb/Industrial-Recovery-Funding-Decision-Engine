# Core Architecture Contracts v0.1

This document defines conceptual contracts. Concrete schemas will be versioned before implementation.

## EvaluationStep

```text
evaluate(
  EvaluationInputBundleV1
) -> EvaluationStepResultV1
```

EvaluationInputBundle references immutable snapshots and policy versions.

A step must not depend on downstream modules.

## ImpactEstimator

```text
estimate(
  ImpactEstimationInputV1
) -> ImpactEstimateV1
```

Output may include:

- expected_jobs
- expected_production
- expected_restart_time
- expected_impact
- risk
- confidence/quality metadata
- estimator_name
- estimator_version

Implementations:

- RuleBasedImpactEstimator — MVP
- MLImpactEstimator — future

## CandidateContract

A solver-neutral candidate representation.

Must contain only domain values required by the optimization problem, plus stable identifiers/version references.

It must not contain OR-Tools classes or solver-specific objects.

## PortfolioOptimizer

```text
optimize(
  PortfolioProblemV1
) -> PortfolioSolutionV1
```

PortfolioProblem contains:

- candidate set reference/content
- budget snapshot
- objective specification
- constraint specification
- reproducibility parameters

PortfolioSolution contains:

- selected candidate IDs
- recommended amounts
- objective value
- constraint status
- solution status
- solver metadata reference

## RecommendationPackage

Human-facing/exported contract.

Consumes exported evaluation and optimization results.

Must not inspect solver internal state.

## AuditEventEnvelope

Cross-cutting safe event envelope:

- event_id
- event_type
- entity_reference
- actor_reference
- correlation_id
- occurred_at
- schema_version
- safe_metadata
- integrity metadata

## OfflineOperation

Queued operation envelope:

- operation_id / idempotency_key
- actor/session reference
- resource reference
- base_version
- operation_type
- payload schema version
- created_at
- client/device reference
- local status

The server validates current authorization and conflict state before applying it.
