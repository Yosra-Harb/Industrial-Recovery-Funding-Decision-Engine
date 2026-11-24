# Product Scope

## Product Goal

Automate the operational decision workflow for industrial recovery funding and optimize the allocation of a limited programme budget across eligible Factory–Intervention combinations.

## MVP In Scope

### Inputs

- Factory master profile/reference
- Recovery application
- Funding provider
- Funding programme
- Portfolio strategy
- Programme constraints
- Intervention catalogue

### Automated Processing

- completeness validation
- data validation
- eligibility screening
- recovery-needs diagnosis
- intervention matching
- impact/priority assessment
- candidate generation
- constrained portfolio optimization
- scenario comparison
- explanation generation

### Human Governance

- review queue
- approve/reject/defer
- request more information
- authorized override with mandatory reason
- audit trail

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
- production-grade ML trained on invented data
- causal treatment-effect estimation
- multi-country policy packs
- advanced supply-chain graph optimization
- full microservices deployment

## Success Criteria for MVP

The MVP must demonstrate:

1. end-to-end processing of a controlled dataset
2. zero funding of explicitly ineligible cases
3. no budget/constraint violation in accepted optimization results
4. reproducible optimization runs
5. explanations for consequential recommendations
6. human override auditability
7. module-level regression protection
8. ability to change programme configuration without editing core business code
