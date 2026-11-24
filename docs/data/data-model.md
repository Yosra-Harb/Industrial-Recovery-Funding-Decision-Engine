# Domain Data Model v0.3

This is a domain model, not the final SQL schema.

Entities are grouped by bounded context. Persistence ownership follows context boundaries.

## Multi-Tenant Rule

The system supports multiple institutions with strict isolation.

A **Tenant** is the primary operational/security boundary. An **Organization** is a legal/institutional participant.

Every business entity below is tenant-scoped unless explicitly described as global control-plane data.

Cross-tenant access is denied by default.

## 1. Identity & Access

### Organization

Global control-plane record.

- organization_id
- organization_name
- organization_type
- status
- external_reference

### Tenant

Global control-plane record identifying an isolated workspace.

- tenant_id
- tenant_name
- tenant_type
- status

### TenantOrganizationMembership

- tenant_id
- organization_id
- relationship_code
- valid_from
- valid_to

### ActorTenantMembership

- membership_id
- actor_id
- tenant_id
- organization_id
- status
- valid_from
- valid_to


Identity details may live in an external identity provider.

Internal references:

### ActorReference

- actor_id
- external_subject
- organization_id
- status

### AuthorizationAssignment

- assignment_id
- actor_id
- role
- organization_scope
- programme_scope
- factory_scope_or_assignment
- valid_from
- valid_to

## 2. Factory Registry

### Factory

Represents stable enterprise master data or an external reference.

- factory_id
- external_reference
- factory_name
- sector
- location
- factory_size
- products
- employees_baseline
- production_capacity_baseline
- registration_status
- year_established
- source_reference
- verification_status

## 3. Programme Catalogue & Policy

### FundingProvider

- provider_id
- provider_name
- provider_type
- external_reference

### FundingProgramme

Stable programme identity.

- programme_id
- provider_id
- programme_name
- status
- currency
- start_date
- end_date

### ProgrammePolicyVersion

Immutable published policy snapshot.

- policy_version_id
- programme_id
- version
- status
- funding_instrument
- total_budget_definition
- max_funding_per_factory
- eligibility_configuration
- required_evidence_configuration
- objective_configuration
- constraint_configuration
- risk_configuration
- fairness_configuration
- created_by
- approved_by
- published_at
- content_hash

### PortfolioStrategy

- strategy_id
- code
- name
- objective_definition
- weights_configuration
- version

### Intervention

- intervention_id
- code
- name
- category
- description
- min_cost
- max_cost
- applicability_configuration
- version

## 4. Application Intake & Validation

### RecoveryApplication

Mutable while draft; submitted versions are snapshotted for evaluation.

- application_id
- factory_id
- programme_id
- current_employees
- current_production_percentage
- operational_status
- damage_level
- building_damage
- equipment_damage
- inventory_damage
- energy_availability
- raw_material_availability
- market_access
- workforce_availability
- working_capital_status
- requested_amount
- requested_intervention
- estimated_restart_cost
- estimated_restart_time
- expected_jobs_restored
- expected_production_recovery
- submission_status
- submitted_at
- row_version

### EvidenceRecord

Metadata/reference to evidence; binary storage design is separate.

- evidence_id
- application_id
- evidence_type
- storage_reference
- classification
- verification_status
- source
- collected_at
- content_hash

### ApplicationSnapshot

Immutable evaluation input.

- application_snapshot_id
- application_id
- snapshot_version
- content_hash
- created_at

## 5. Evaluation Run

### EvaluationRun

- evaluation_run_id
- application_snapshot_id
- policy_version_id
- indicator_catalog_version
- impact_estimator_name
- impact_estimator_version
- started_at
- completed_at
- status
- correlation_id

### EvaluationStepResult

Generic persisted result envelope for replaceable steps.

- step_result_id
- evaluation_run_id
- step_code
- step_contract_version
- status
- result_reference_or_payload
- explanation_reference
- created_at

### AssessmentResult

Decision-oriented exported assessment.

- assessment_result_id
- evaluation_run_id
- validation_status
- eligibility_status
- primary_bottleneck
- restart_readiness
- jobs_impact
- production_impact
- cost_efficiency
- risk
- priority
- expected_impact
- needs_human_review
- review_reason

### InterventionCandidate

- candidate_id
- evaluation_run_id
- intervention_id
- required_cost
- expected_jobs
- expected_production
- expected_restart_time
- expected_impact
- risk
- candidate_contract_version

### CandidateSetSnapshot

- candidate_set_id
- programme_id
- policy_version_id
- candidate_count
- content_hash
- created_at

## 6. Portfolio Optimization

### OptimizationRun

- optimization_run_id
- programme_id
- policy_version_id
- strategy_id
- candidate_set_id
- budget_snapshot_id
- problem_spec_version
- optimizer_adapter_name
- optimizer_adapter_version
- solver_name
- solver_version
- solver_parameters
- random_seed
- thread_count
- time_limit
- optimality_gap
- input_hash
- started_at
- completed_at
- status
- objective_value

### OptimizationResult

- optimization_result_id
- optimization_run_id
- candidate_id
- selected
- recommended_amount
- marginal_impact
- selection_reason_code

## 7. Decision & Governance

### RecommendationPackage

Solver-independent human-review package.

- recommendation_id
- optimization_run_id
- explanation_version
- generated_at
- content_hash

### HumanDecision

- decision_id
- application_id
- recommendation_id
- decision
- actor_id
- reason
- is_override
- conflict_of_interest_declared
- decided_at

### BudgetSnapshot

- budget_snapshot_id
- programme_id
- policy_version_id
- authorized_budget
- committed_amount
- available_amount
- captured_at
- content_hash

### BudgetCommitment

- commitment_id
- programme_id
- decision_id
- amount
- currency
- status
- committed_at
- released_at
- row_version

## 8. Monitoring & Outcomes

### MonitoringOutcome

- outcome_id
- application_id
- approved_amount
- actual_intervention
- funding_date
- actual_restart_date
- actual_restart_time
- actual_jobs_restored
- actual_production_percentage
- actual_cost
- intervention_completion_percentage
- operational_status
- measured_at
- evidence_reference

## Cross-Cutting Audit & Provenance

### AuditEvent

- audit_event_id
- entity_type
- entity_id
- event_type
- actor_reference
- correlation_id
- configuration_version
- previous_event_hash
- event_hash
- timestamp
- safe_metadata

## Relationship Principles

Priority is not a permanent Factory attribute.

```text
Factory
+ Application Snapshot
+ Programme Policy Version
+ Evaluation Run
→ Candidate(s)
→ Optimization
→ Recommendation
→ Human Decision
→ Budget Commitment
```

Published programme policy versions and historical decision input snapshots are immutable.

The final SQL schema must preserve context ownership and must not create cross-context foreign-key coupling that defeats module independence without explicit justification.


## Tenant Ownership Principle

Factory, Programme, Application, Evaluation, Optimization, Decision, Monitoring and tenant audit records all carry `tenant_id`.

Tenant-local relationships must not point to records owned by another tenant.

A joint institutional programme should be represented by a dedicated tenant/workspace rather than arbitrary cross-tenant sharing.
