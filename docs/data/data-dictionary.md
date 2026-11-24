# Data Dictionary Standards v0.2

This file defines naming, typing and data-governance conventions.

## Naming

- internal identifiers: UUID
- backend/database fields: snake_case
- timestamps: UTC `timestamptz`
- date-only values: PostgreSQL `date`
- currencies: ISO 4217 `char(3)`
- stable status codes: uppercase string codes
- avoid PostgreSQL ENUM until stable migration requirements justify it

## Numeric Representation

### Money

```text
numeric(18,2)
```

Never use floating point for money.

### Ratios / Percentages

Normalized ratios use:

```text
numeric(6,5)
```

with valid range `0 <= value <= 1`.

UI layers may render these as 0–100%.

### Scores

A score must document its scale.

Do not assume every score is 0–1.

## Identifiers

All public/internal aggregate identifiers are UUIDs.

The exact generation standard (UUIDv7 vs UUIDv4) will be finalized before executable migrations.

## Data Classification

Initial classes:

- PUBLIC
- INTERNAL
- CONFIDENTIAL
- RESTRICTED

Real enterprise operational, funding, ownership, contact and evidence data is expected to be at least CONFIDENTIAL unless deployment policy defines otherwise.

## Null Semantics

Missing values must not silently become zero.

Preserve the distinction between:

- UNKNOWN
- NOT_APPLICABLE
- ZERO
- NOT_COLLECTED

When this distinction affects decision logic, use an explicit status/code rather than relying only on SQL NULL.

## Mutable Data

Mutable aggregate roots use:

- `created_at`
- `updated_at`
- `row_version`

The application must use optimistic concurrency where simultaneous/offline edits are possible.

## Immutable Decision Data

Decision-critical snapshots use:

- immutable payload/typed fields
- schema/version reference
- content hash
- creation timestamp

Updates create a new version/run/snapshot.

## Flexible JSONB

JSONB is permitted for intentionally extensible configuration or result envelopes.

JSONB must not hide core invariants that require database/application validation.

Every important JSONB payload has:

- schema version
- owner context
- validation model
- canonical hashing rule where used in decision reproduction

## Required Provenance Metadata

Decision-relevant data should preserve as appropriate:

- source
- collection time
- verification status
- schema/version
- actor/system origin
- content hash
- correlation/run reference

## Derived Fields

Derived metrics document:

- formula/algorithm
- input contract
- estimator version
- valid range
- interpretation
- whether higher/lower is better
- missing-data behavior

## Evidence

Evidence database rows contain metadata and secure storage references.

Raw binary evidence should live behind an access-controlled object-storage abstraction rather than inside ordinary relational rows unless a later ADR says otherwise.

## Synthetic Data

Synthetic/demo data must be explicitly marked.

Synthetic data must never be represented as real beneficiary or field-validation data.

## Cross-Context References

Cross-context IDs are soft references by default.

The owning application service validates existence and authorization before authoritative operations.

A database-level cross-context FK requires explicit architectural justification.

## Retention

Retention period and disposal behavior must eventually be defined per data classification and deployment policy.

No module may assume indefinite retention of PII.
