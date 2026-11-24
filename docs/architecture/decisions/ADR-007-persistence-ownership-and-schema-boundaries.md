# ADR-007: Persistence Ownership and PostgreSQL Schema Boundaries

- Status: Accepted
- Date: 2026-11-24

## Context

The project requires strict module independence while starting as a Modular Monolith on PostgreSQL.

A single undifferentiated database schema would allow modules to bypass contracts and read/write each other's tables directly, creating hidden coupling.

At the same time, introducing a separate database per module at MVP stage would add unnecessary operational complexity.

## Decision

Use **one PostgreSQL database with separate PostgreSQL schemas per bounded context**.

Initial schemas:

- `iam`
- `factory`
- `programme`
- `application`
- `evaluation`
- `optimization`
- `decision`
- `monitoring`
- `audit`

Each bounded context owns its PostgreSQL schema and is the only context allowed to mutate its tables.

## Foreign-Key Policy

### Inside one bounded context

Foreign keys are required where they protect valid internal relationships.

### Across bounded contexts

Cross-context foreign keys are **disallowed by default**.

Cross-context references use stable UUID identifiers and are validated through application contracts.

An exception requires explicit architectural justification and must not create reverse dependency or shared write ownership.

## Database Roles

Production-capable deployments should use context-scoped database roles where operationally feasible.

A context should receive:

- read/write on its own schema
- no direct write access to other schemas
- only explicitly approved read access/read models where required

Administrative migration credentials are separate from application runtime credentials.

## Reporting

Cross-context reporting must use:

- approved read models
- materialized views owned by a reporting boundary
- application-level aggregation
- or event-fed analytical stores later

Business modules must not create ad-hoc joins against private tables of other contexts.

## Mutable vs Immutable Records

Mutable operational records use optimistic concurrency, typically a `row_version`.

Decision-critical snapshots and published policy versions are immutable.

Examples of immutable records:

- published ProgrammePolicyVersion
- ApplicationSnapshot
- CandidateSetSnapshot
- official OptimizationRun inputs/metadata
- RecommendationPackage content snapshot
- AuditEvent

## JSONB Policy

JSONB is allowed only for configuration or versioned payloads that are intentionally extensible.

Core invariants and frequently queried decision fields should remain typed columns.

Every JSONB configuration payload must include or reference a schema/version and must be validated by the owning context.

## Consequences

### Positive

- preserves module ownership inside one deployable database
- avoids premature database-per-service complexity
- makes unauthorized cross-module persistence access visible
- supports later service/database extraction
- enables context-level migration ownership

### Negative

- referential integrity across contexts is not automatically enforced by cross-context foreign keys
- application contracts and integrity tests become important
- database privileges and migration ownership require discipline

## Future Extraction

If a bounded context is extracted into a service later, its schema is a natural migration boundary.

Public contracts should remain stable during extraction where feasible.
