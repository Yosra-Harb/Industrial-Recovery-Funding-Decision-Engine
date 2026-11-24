# Product Scope v0.2

## Product Goal

Automate industrial recovery funding decision workflows and optimize limited programme budgets across eligible Factory–Intervention combinations while preserving human authority and full reproducibility.

## MVP In Scope

### Inputs

- factory master reference
- recovery application
- evidence metadata
- funding provider
- published programme policy version
- portfolio strategy
- budget snapshot
- intervention catalogue

### Automated Processing

- completeness/data validation
- eligibility screening
- recovery-needs diagnosis
- intervention matching
- indicator generation
- rule-based impact estimation
- candidate generation
- constrained portfolio optimization
- scenario comparison
- explanation generation

### Governance

- review queue
- approve/reject/defer
- request more information
- override with mandatory reason
- four-eyes publication/approval where configured
- budget commitment protection
- audit/provenance

### Offline MVP

- draft application capture/edit
- cached reference data
- queued evidence metadata
- queued synchronization
- conflict detection
- idempotent replay

Authoritative funding decisions remain online-only.

### Monitoring Prototype

- funded intervention
- approved amount
- actual restart
- jobs restored
- production recovery
- actual cost
- expected-vs-actual comparison

## Explicitly Out of Scope for MVP

- autonomous final funding decisions
- money transfer/payment execution
- banking ledger
- production-grade predictive ML
- causal treatment-effect estimation
- multi-country policy packs
- advanced supply-chain graph optimization
- full microservices deployment
- official optimization/final approvals while offline

## MVP Success Criteria

1. end-to-end processing of a controlled dataset
2. zero selected ineligible cases
3. no accepted budget/constraint violations
4. historical decision reproducibility from recorded snapshots/versions
5. explanations for consequential recommendations
6. auditable overrides
7. no budget oversubscription under concurrent approval tests
8. module-level regression protection
9. programme policy changes without core code edits
10. authorization scope tests for factory/programme/organization access
11. safe offline sync with conflict detection and idempotency
