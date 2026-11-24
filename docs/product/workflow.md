# End-to-End Workflow v0.2

## Primary Data Flow

```text
Factory Master Profile / Reference
        +
Recovery Application + Evidence Metadata
        +
Published Programme Policy Version
        +
Portfolio Strategy
        +
Budget Snapshot
        ↓
Application Intake & Validation
        ↓
Evaluation Run
        ├─ Eligibility Step
        ├─ Recovery Diagnosis Step
        ├─ Intervention Matching Step
        ├─ Indicator Generation Step
        └─ Impact Estimation Step
        ↓
Versioned Candidate Factory–Intervention Set
        ↓
Portfolio Optimization Run
        ↓
Recommendation Package
        ↓
Human Review / Approval
        ↓
Decision Record
        ↓
Budget Commitment
        ↓
Post-Funding Monitoring
        ↓
Outcome Observations
        ↓
Expected vs Actual Analysis
        ↓
Learning Dataset / Analytics
        ↓
Future Predictive Models
        ↓
Future Re-optimization
```

## Automation Principle

The intended operating model is **automation-first, exception-driven human review**.

Routine cases may proceed automatically where programme policy allows.

Human attention is prioritized for incomplete evidence, conflicts, policy exceptions, high-risk/high-value cases, final governance approval and overrides.

## Evaluation Run

An Evaluation Run is the authoritative orchestration record for the decision pipeline.

It references:

- immutable application snapshot
- programme policy version
- indicator definitions
- estimator version
- each step result and status

Pipeline steps are replaceable behind contracts and must not access downstream implementation details.

## Recommendation Package

The recommendation package is solver-independent and contains:

- selected/not-selected status
- recommended intervention
- recommended amount
- reasons/evidence references
- risk/impact indicators
- optimization run reference
- policy/version references

The explanation layer consumes exported results and must not inspect solver internals directly.

## Budget Authority

Optimization proposes a portfolio against a budget snapshot.

Only Decision & Governance creates authoritative Budget Commitments.

This prevents concurrent approvals from oversubscribing the programme.

## State Model — Initial Draft

```text
DRAFT
→ SUBMITTED
→ VALIDATING
→ INCOMPLETE | READY
→ EVALUATING
→ INELIGIBLE | NEEDS_REVIEW | ELIGIBLE
→ CANDIDATE_READY
→ OPTIMIZATION_READY
→ RECOMMENDED
→ COMMITTEE_REVIEW
→ APPROVED | REJECTED | DEFERRED
→ COMMITTED
→ FUNDED
→ MONITORING
→ COMPLETED
```

Every consequential transition must be auditable.

## Offline Authority Boundary

Offline:

- draft capture/edit
- cached reference data
- queued evidence metadata/uploads
- non-authoritative local validation
- queued sync

Online authoritative only:

- programme publication
- official evaluation finalization
- official optimization run
- approval/rejection/override
- budget commitment

Offline operations must carry idempotency keys and base-version references for conflict detection.
