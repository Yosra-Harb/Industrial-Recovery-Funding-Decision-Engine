# Data Dictionary Standards

This file defines naming and data-governance conventions. Detailed field constraints will be added before database migrations are created.

## Naming

- internal identifiers: UUID preferred
- snake_case for backend/database field names
- timestamps: UTC, ISO 8601
- currencies: ISO 4217 code + decimal amount
- percentages: document whether represented as 0–1 or 0–100; do not mix
- enums: uppercase stable codes

## Data Classification

Initial classes:

- PUBLIC
- INTERNAL
- CONFIDENTIAL
- RESTRICTED

Real enterprise operational, funding, ownership, contact, and evidence data is expected to be at least CONFIDENTIAL unless policy defines otherwise.

## Required Metadata for Decision Inputs

Decision-relevant data should preserve:

- source
- collection time
- validation status
- version
- actor/system origin where appropriate

## Missing Data

Missing values must not be silently converted to zero.

The distinction between:

- UNKNOWN
- NOT_APPLICABLE
- ZERO
- NOT_COLLECTED

must be preserved.

## Derived Fields

Derived metrics must document:

- formula/algorithm
- input fields
- version
- valid range
- interpretation

## Synthetic Data

Synthetic/demo data must be explicitly marked and must never be represented as real beneficiary data.

## Next Step

The next revision will define types, nullability, validation ranges, enum values, and ownership module for every MVP field.
