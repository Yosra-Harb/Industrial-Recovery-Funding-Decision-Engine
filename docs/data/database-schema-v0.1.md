# Database Schema v0.1

## Status

**Logical/physical design baseline before migrations.**

Database: PostgreSQL  
Architecture: one database, separate schema per bounded context.

This document defines table ownership, primary fields, internal constraints and initial indexes.

It is not yet an executable migration.

---

# 1. Global Conventions

## Identifiers

Use UUID primary keys.

Preferred application-generated identifiers: UUIDv7 when the selected runtime/library support is approved; otherwise UUIDv4.

## Time

Use `timestamptz` and store timestamps in UTC.

## Money

Use:

```text
amount numeric(18,2)
currency char(3)
```

Currency uses ISO 4217 codes.

## Percentages

Use numeric values in the normalized range `0.0000–1.0000` unless a field explicitly documents another scale.

## Mutable Rows

Mutable aggregate roots include:

```text
row_version bigint not null
created_at timestamptz not null
updated_at timestamptz not null
```

`row_version` supports optimistic concurrency and offline conflict detection.

## Immutable Rows

Immutable decision snapshots contain a content hash and are never updated after finalization/publication.

## Flexible Configuration

Use JSONB only for versioned extensible configuration.

Each JSONB configuration requires:

- configuration schema/version
- application validation
- deterministic canonicalization before hashing when content hashes are used

---

# 2. Schema: iam

Owner: **Identity & Access**

## iam.actor_reference

Purpose: internal reference to an authenticated person/service without storing identity-provider credentials.

Fields:

| Field | Type | Null | Notes |
|---|---|---:|---|
| actor_id | uuid PK | no | internal actor ID |
| external_subject | varchar(255) | no | OIDC subject/service subject |
| organization_id | uuid | yes | scoped organization reference |
| actor_type | varchar(32) | no | HUMAN / SERVICE |
| status | varchar(32) | no | ACTIVE / SUSPENDED / DISABLED |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |
| row_version | bigint | no | optimistic concurrency |

Constraints:

- unique(`external_subject`)
- status must use an approved stable code

Indexes:

- `external_subject`
- `organization_id, status`

## iam.authorization_assignment

Purpose: role plus resource scope.

Fields:

| Field | Type | Null |
|---|---|---:|
| assignment_id | uuid PK | no |
| actor_id | uuid FK → iam.actor_reference | no |
| role_code | varchar(64) | no |
| organization_scope | uuid | yes |
| programme_scope | uuid | yes |
| factory_scope | uuid | yes |
| valid_from | timestamptz | no |
| valid_to | timestamptz | yes |
| created_at | timestamptz | no |

Indexes:

- `actor_id`
- `programme_scope`
- `factory_scope`

---

# 3. Schema: factory

Owner: **Factory Registry**

## factory.factory

Fields:

| Field | Type | Null | Notes |
|---|---|---:|---|
| factory_id | uuid PK | no | |
| external_reference | varchar(255) | yes | host-platform reference |
| factory_name | varchar(255) | no | confidential in real deployment |
| sector_code | varchar(64) | no | |
| location_code | varchar(128) | yes | avoid unnecessary precision where not needed |
| factory_size_code | varchar(32) | yes | |
| products | jsonb | yes | structured product list |
| employees_baseline | integer | yes | >= 0 |
| production_capacity_baseline | numeric(18,4) | yes | unit defined by source metadata |
| registration_status | varchar(32) | yes | |
| year_established | smallint | yes | |
| source_reference | varchar(255) | yes | |
| verification_status | varchar(32) | no | UNVERIFIED / PARTIAL / VERIFIED |
| created_at | timestamptz | no | |
| updated_at | timestamptz | no | |
| row_version | bigint | no | |

Constraints:

- employees_baseline >= 0 when present
- year_established uses a valid configured range
- external_reference may be unique per source, not globally assumed

Indexes:

- `sector_code`
- `verification_status`
- approved external reference lookup index

---

# 4. Schema: programme

Owner: **Programme Catalogue & Policy**

## programme.funding_provider

Fields:

| Field | Type | Null |
|---|---|---:|
| provider_id | uuid PK | no |
| provider_name | varchar(255) | no |
| provider_type | varchar(64) | no |
| external_reference | varchar(255) | yes |
| created_at | timestamptz | no |
| updated_at | timestamptz | no |
| row_version | bigint | no |

## programme.funding_programme

Stable programme identity; policy details are versioned separately.

Fields:

| Field | Type | Null |
|---|---|---:|
| programme_id | uuid PK | no |
| provider_id | uuid FK → programme.funding_provider | no |
| programme_name | varchar(255) | no |
| status | varchar(32) | no |
| currency | char(3) | no |
| start_date | date | yes |
| end_date | date | yes |
| created_at | timestamptz | no |
| updated_at | timestamptz | no |
| row_version | bigint | no |

Constraint:

- end_date >= start_date when both exist

Indexes:

- `provider_id, status`

## programme.programme_policy_version

Immutable after PUBLISHED.

Fields:

| Field | Type | Null | Notes |
|---|---|---:|---|
| policy_version_id | uuid PK | no | |
| programme_id | uuid FK → programme.funding_programme | no | |
| version_no | integer | no | monotonic per programme |
| status | varchar(32) | no | DRAFT / PENDING_APPROVAL / PUBLISHED / RETIRED |
| funding_instrument_code | varchar(64) | no | |
| total_budget_definition | numeric(18,2) | no | configured programme ceiling |
| max_funding_per_factory | numeric(18,2) | yes | |
| eligibility_configuration | jsonb | no | versioned rules |
| required_evidence_configuration | jsonb | yes | |
| objective_configuration | jsonb | no | |
| constraint_configuration | jsonb | no | |
| risk_configuration | jsonb | yes | |
| fairness_configuration | jsonb | yes | |
| config_schema_version | varchar(32) | no | |
| created_by | uuid | no | actor reference; cross-context soft ref |
| approved_by | uuid | yes | cross-context soft ref |
| published_at | timestamptz | yes | |
| content_hash | varchar(128) | no | |
| created_at | timestamptz | no | |

Constraints:

- unique(`programme_id, version_no`)
- PUBLISHED requires `approved_by` and `published_at`
- published content is immutable

Indexes:

- `programme_id, status`
- `content_hash`

## programme.portfolio_strategy

Fields:

| Field | Type | Null |
|---|---|---:|
| strategy_id | uuid PK | no |
| code | varchar(64) | no |
| name | varchar(255) | no |
| objective_definition | jsonb | no |
| weights_configuration | jsonb | yes |
| config_schema_version | varchar(32) | no |
| version_no | integer | no |
| content_hash | varchar(128) | no |
| created_at | timestamptz | no |

Constraint:

- unique(`code, version_no`)

## programme.intervention

Versioned intervention catalogue item.

Fields:

| Field | Type | Null |
|---|---|---:|
| intervention_id | uuid PK | no |
| code | varchar(64) | no |
| name | varchar(255) | no |
| category_code | varchar(64) | no |
| description | text | yes |
| min_cost | numeric(18,2) | yes |
| max_cost | numeric(18,2) | yes |
| applicability_configuration | jsonb | yes |
| config_schema_version | varchar(32) | no |
| version_no | integer | no |
| active | boolean | no |
| created_at | timestamptz | no |

Constraints:

- min_cost >= 0 when present
- max_cost >= min_cost when both exist
- unique(`code, version_no`)

---

# 5. Schema: application

Owner: **Application Intake & Validation**

Cross-context IDs such as `factory_id` and `programme_id` are soft references, not database foreign keys by default.

## application.recovery_application

Fields:

| Field | Type | Null |
|---|---|---:|
| application_id | uuid PK | no |
| factory_id | uuid | no |
| programme_id | uuid | no |
| current_employees | integer | yes |
| current_production_ratio | numeric(6,5) | yes |
| operational_status | varchar(32) | no |
| damage_level_code | varchar(32) | yes |
| building_damage_ratio | numeric(6,5) | yes |
| equipment_damage_ratio | numeric(6,5) | yes |
| inventory_damage_ratio | numeric(6,5) | yes |
| energy_availability_code | varchar(32) | yes |
| raw_material_availability_code | varchar(32) | yes |
| market_access_code | varchar(32) | yes |
| workforce_availability_code | varchar(32) | yes |
| working_capital_status_code | varchar(32) | yes |
| requested_amount | numeric(18,2) | yes |
| requested_intervention_code | varchar(64) | yes |
| estimated_restart_cost | numeric(18,2) | yes |
| estimated_restart_days | integer | yes |
| expected_jobs_restored | integer | yes |
| expected_production_recovery_ratio | numeric(6,5) | yes |
| submission_status | varchar(32) | no |
| submitted_at | timestamptz | yes |
| created_at | timestamptz | no |
| updated_at | timestamptz | no |
| row_version | bigint | no |

Constraints:

- numeric ratios in [0,1]
- money/count/day values >= 0 when present

Indexes:

- `factory_id`
- `programme_id, submission_status`
- `submitted_at`

## application.evidence_record

Fields:

| Field | Type | Null |
|---|---|---:|
| evidence_id | uuid PK | no |
| application_id | uuid FK → application.recovery_application | no |
| evidence_type_code | varchar(64) | no |
| storage_reference | text | no |
| classification_code | varchar(32) | no |
| verification_status | varchar(32) | no |
| source_code | varchar(64) | yes |
| collected_at | timestamptz | yes |
| content_hash | varchar(128) | no |
| created_at | timestamptz | no |

No raw binary evidence is stored directly in this table.

## application.application_snapshot

Immutable snapshot.

Fields:

| Field | Type | Null |
|---|---|---:|
| application_snapshot_id | uuid PK | no |
| application_id | uuid FK → application.recovery_application | no |
| snapshot_version | integer | no |
| snapshot_payload | jsonb | no |
| snapshot_schema_version | varchar(32) | no |
| content_hash | varchar(128) | no |
| created_at | timestamptz | no |

Constraints:

- unique(`application_id, snapshot_version`)
- immutable after insert

---

# 6. Schema: evaluation

Owner: **Evaluation Run**

## evaluation.evaluation_run

Fields:

| Field | Type | Null |
|---|---|---:|
| evaluation_run_id | uuid PK | no |
| application_snapshot_id | uuid | no |
| policy_version_id | uuid | no |
| indicator_catalog_version | varchar(64) | no |
| impact_estimator_name | varchar(128) | no |
| impact_estimator_version | varchar(64) | no |
| started_at | timestamptz | no |
| completed_at | timestamptz | yes |
| status | varchar(32) | no |
| correlation_id | uuid | no |
| input_hash | varchar(128) | no |

Indexes:

- `application_snapshot_id`
- `policy_version_id`
- `status, started_at`

## evaluation.evaluation_step_result

Fields:

| Field | Type | Null |
|---|---|---:|
| step_result_id | uuid PK | no |
| evaluation_run_id | uuid FK → evaluation.evaluation_run | no |
| step_code | varchar(64) | no |
| step_contract_version | varchar(32) | no |
| status | varchar(32) | no |
| result_payload | jsonb | no |
| result_schema_version | varchar(32) | no |
| result_hash | varchar(128) | no |
| explanation_reference | text | yes |
| created_at | timestamptz | no |

Constraint:

- unique(`evaluation_run_id, step_code`) for the MVP unless retries are modeled separately

## evaluation.assessment_result

Typed exported assessment.

Fields:

| Field | Type | Null |
|---|---|---:|
| assessment_result_id | uuid PK | no |
| evaluation_run_id | uuid FK → evaluation.evaluation_run | no |
| validation_status | varchar(32) | no |
| eligibility_status | varchar(32) | no |
| primary_bottleneck_code | varchar(64) | yes |
| restart_readiness | numeric(6,5) | yes |
| jobs_impact | numeric(18,4) | yes |
| production_impact | numeric(18,4) | yes |
| cost_efficiency | numeric(18,6) | yes |
| risk_score | numeric(6,5) | yes |
| priority_score | numeric(18,6) | yes |
| expected_impact | numeric(18,6) | yes |
| needs_human_review | boolean | no |
| review_reason_code | varchar(64) | yes |
| result_hash | varchar(128) | no |
| created_at | timestamptz | no |

Constraint:

- unique(`evaluation_run_id`)

## evaluation.intervention_candidate

Fields:

| Field | Type | Null |
|---|---|---:|
| candidate_id | uuid PK | no |
| evaluation_run_id | uuid FK → evaluation.evaluation_run | no |
| intervention_id | uuid | no |
| required_cost | numeric(18,2) | no |
| expected_jobs | numeric(18,4) | yes |
| expected_production | numeric(18,6) | yes |
| expected_restart_days | numeric(18,4) | yes |
| expected_impact | numeric(18,6) | no |
| risk_score | numeric(6,5) | yes |
| candidate_contract_version | varchar(32) | no |
| content_hash | varchar(128) | no |
| created_at | timestamptz | no |

Indexes:

- `evaluation_run_id`
- `intervention_id`

## evaluation.candidate_set_snapshot

Fields:

| Field | Type | Null |
|---|---|---:|
| candidate_set_id | uuid PK | no |
| programme_id | uuid | no |
| policy_version_id | uuid | no |
| candidate_count | integer | no |
| content_hash | varchar(128) | no |
| created_at | timestamptz | no |

## evaluation.candidate_set_item

Membership table.

Fields:

| Field | Type | Null |
|---|---|---:|
| candidate_set_id | uuid FK → evaluation.candidate_set_snapshot | no |
| candidate_id | uuid FK → evaluation.intervention_candidate | no |
| ordinal_no | integer | no |

Primary key:

- (`candidate_set_id, candidate_id`)

---

# 7. Schema: decision

Owner: **Decision & Governance**

## decision.budget_snapshot

Immutable snapshot of decision-authoritative budget state.

Fields:

| Field | Type | Null |
|---|---|---:|
| budget_snapshot_id | uuid PK | no |
| programme_id | uuid | no |
| policy_version_id | uuid | no |
| authorized_budget | numeric(18,2) | no |
| committed_amount | numeric(18,2) | no |
| available_amount | numeric(18,2) | no |
| currency | char(3) | no |
| captured_at | timestamptz | no |
| content_hash | varchar(128) | no |

Invariant:

```text
available_amount = authorized_budget - committed_amount
```

## decision.recommendation_package

Immutable human-review package.

Fields:

| Field | Type | Null |
|---|---|---:|
| recommendation_id | uuid PK | no |
| optimization_run_id | uuid | no |
| explanation_version | varchar(64) | no |
| package_payload | jsonb | no |
| package_schema_version | varchar(32) | no |
| content_hash | varchar(128) | no |
| generated_at | timestamptz | no |

## decision.human_decision

Fields:

| Field | Type | Null |
|---|---|---:|
| decision_id | uuid PK | no |
| application_id | uuid | no |
| recommendation_id | uuid FK → decision.recommendation_package | no |
| decision_code | varchar(32) | no |
| actor_id | uuid | no |
| reason_text | text | yes |
| is_override | boolean | no |
| conflict_of_interest_declared | boolean | no |
| decided_at | timestamptz | no |
| decision_hash | varchar(128) | no |

## decision.budget_commitment

Mutable only through controlled commitment/release operations.

Fields:

| Field | Type | Null |
|---|---|---:|
| commitment_id | uuid PK | no |
| programme_id | uuid | no |
| decision_id | uuid FK → decision.human_decision | no |
| amount | numeric(18,2) | no |
| currency | char(3) | no |
| status | varchar(32) | no |
| committed_at | timestamptz | no |
| released_at | timestamptz | yes |
| row_version | bigint | no |

Critical invariant:

- commitment creation/release must use a transaction that prevents programme budget oversubscription

Indexes:

- `programme_id, status`
- unique active commitment policy to be finalized before migration

---

# 8. Schema: optimization

Owner: **Portfolio Optimization**

## optimization.optimization_run

Fields:

| Field | Type | Null |
|---|---|---:|
| optimization_run_id | uuid PK | no |
| programme_id | uuid | no |
| policy_version_id | uuid | no |
| strategy_id | uuid | no |
| candidate_set_id | uuid | no |
| budget_snapshot_id | uuid | no |
| problem_spec_version | varchar(32) | no |
| optimizer_adapter_name | varchar(128) | no |
| optimizer_adapter_version | varchar(64) | no |
| solver_name | varchar(128) | no |
| solver_version | varchar(64) | no |
| solver_parameters | jsonb | no |
| random_seed | bigint | yes |
| thread_count | integer | yes |
| time_limit_seconds | numeric(12,3) | yes |
| optimality_gap | numeric(12,8) | yes |
| input_hash | varchar(128) | no |
| started_at | timestamptz | no |
| completed_at | timestamptz | yes |
| status | varchar(32) | no |
| objective_value | numeric(24,8) | yes |

Indexes:

- `programme_id, started_at`
- `candidate_set_id`
- `input_hash`

## optimization.optimization_result

Fields:

| Field | Type | Null |
|---|---|---:|
| optimization_result_id | uuid PK | no |
| optimization_run_id | uuid FK → optimization.optimization_run | no |
| candidate_id | uuid | no |
| selected | boolean | no |
| recommended_amount | numeric(18,2) | yes |
| marginal_impact | numeric(18,8) | yes |
| selection_reason_code | varchar(64) | yes |
| created_at | timestamptz | no |

Constraint:

- unique(`optimization_run_id, candidate_id`)

---

# 9. Schema: monitoring

Owner: **Monitoring & Outcomes**

## monitoring.monitoring_outcome

Fields:

| Field | Type | Null |
|---|---|---:|
| outcome_id | uuid PK | no |
| application_id | uuid | no |
| approved_amount | numeric(18,2) | yes |
| actual_intervention_code | varchar(64) | yes |
| funding_date | date | yes |
| actual_restart_date | date | yes |
| actual_restart_days | integer | yes |
| actual_jobs_restored | integer | yes |
| actual_production_ratio | numeric(6,5) | yes |
| actual_cost | numeric(18,2) | yes |
| intervention_completion_ratio | numeric(6,5) | yes |
| operational_status | varchar(32) | yes |
| measured_at | timestamptz | no |
| evidence_reference | text | yes |
| created_at | timestamptz | no |

Indexes:

- `application_id, measured_at`

Multiple observations per application are allowed.

---

# 10. Schema: audit

Owner: **Audit & Provenance**

## audit.audit_event

Append-only/tamper-evident event record.

Fields:

| Field | Type | Null |
|---|---|---:|
| audit_event_id | uuid PK | no |
| entity_type | varchar(128) | no |
| entity_id | uuid | no |
| event_type | varchar(128) | no |
| actor_reference | uuid | yes |
| correlation_id | uuid | no |
| configuration_version | varchar(64) | yes |
| schema_version | varchar(32) | no |
| previous_event_hash | varchar(128) | yes |
| event_hash | varchar(128) | no |
| occurred_at | timestamptz | no |
| safe_metadata | jsonb | yes |

Rules:

- application runtime cannot UPDATE or DELETE audit events
- event hash generation uses a canonical event representation
- PII must not be copied into safe_metadata without explicit justification

Indexes:

- `entity_type, entity_id, occurred_at`
- `correlation_id`
- `occurred_at`

Large-scale partitioning is deferred until workload requires it.

---

# 11. Cross-Context Reference Matrix

These IDs are intentionally soft references unless an ADR approves a database FK.

| Owner table | External reference | Target context |
|---|---|---|
| application.recovery_application | factory_id | Factory Registry |
| application.recovery_application | programme_id | Programme Catalogue |
| programme.programme_policy_version | created_by / approved_by | Identity & Access |
| evaluation.evaluation_run | application_snapshot_id | Application |
| evaluation.evaluation_run | policy_version_id | Programme |
| evaluation.intervention_candidate | intervention_id | Programme |
| evaluation.candidate_set_snapshot | programme_id / policy_version_id | Programme |
| decision.budget_snapshot | programme_id / policy_version_id | Programme |
| decision.human_decision | application_id / actor_id | Application / IAM |
| decision.budget_commitment | programme_id | Programme |
| optimization.optimization_run | programme/policy/strategy/candidate/budget IDs | multiple contexts |
| optimization.optimization_result | candidate_id | Evaluation |
| monitoring.monitoring_outcome | application_id | Application |

Application services validate these references before authoritative operations.

---

# 12. Immutability Matrix

| Record | Mutable? | Rule |
|---|---:|---|
| factory.factory | yes | optimistic concurrency |
| programme.funding_programme | yes | stable identity metadata |
| programme.programme_policy_version DRAFT | yes | until publication |
| programme.programme_policy_version PUBLISHED | no | immutable |
| application.recovery_application DRAFT | yes | optimistic concurrency |
| application.application_snapshot | no | immutable |
| evaluation.evaluation_run | execution state only | inputs immutable |
| evaluation step/result records | no after completion | append/new run for change |
| candidate set snapshot | no | immutable |
| budget snapshot | no | immutable |
| optimization run/result | no after completion | new run for re-optimization |
| recommendation package | no | immutable |
| human decision | no | correction creates a new governed decision event |
| budget commitment | controlled state transitions | no arbitrary updates |
| monitoring outcome | no | new observation for correction/update |
| audit event | no | append-only |

---

# 13. Database Access Rules

Runtime database access must follow context ownership.

At minimum, code review and architecture tests enforce:

- no module imports another context's repository implementation
- no raw SQL joining private tables across contexts
- no business module writes `audit` tables directly
- no optimization code updates `decision.budget_commitment`
- no UI accesses PostgreSQL directly

Context-scoped PostgreSQL roles are recommended for production deployments.

---

# 14. Open Decisions Before Executable Migration

These must be resolved before the first migration is treated as stable:

1. approved UUID implementation (UUIDv7 vs UUIDv4)
2. exact stable status-code catalog
3. canonical hashing algorithm/serialization format
4. evidence binary/object storage provider abstraction
5. exact transaction/locking strategy for BudgetCommitment
6. multi-tenancy key model if more than one implementing organization shares one deployment
7. retention rules per data classification
8. whether any cross-context stable identity reference deserves an explicit database FK exception

None of these block the logical schema review, but they block calling the first SQL migration production-ready.
