# ADR-001: Start with a Modular Monolith

- Status: Accepted
- Date: 2026-11-24

## Context

The engine must be delivered quickly while preserving strict module independence, high security, offline-capable workflows, and future scalability.

Starting with distributed microservices would increase deployment, observability, networking, security, and consistency complexity before workload evidence justifies it.

A traditional unstructured monolith would create unacceptable coupling.

## Decision

Use a **Modular Monolith with strict domain boundaries** for the initial system.

Each module has:

- owned domain logic
- owned application services
- explicit interfaces/contracts
- controlled infrastructure dependencies
- independent tests
- controlled persistence ownership

Cross-module interaction must occur through explicit application contracts, not internal implementation access.

## Consequences

### Positive

- faster MVP delivery
- easier local deployment
- simpler transactional consistency
- easier offline-oriented development
- lower infrastructure burden
- strong testability
- future service extraction remains possible

### Negative

- architectural boundaries must be actively enforced
- poor discipline could degrade into a tightly coupled monolith
- some independent scaling requires later extraction

## Extraction Rule

A module may later become an independent service when justified by:

- independent scaling needs
- security/isolation needs
- deployment cadence
- computational workload
- team ownership
- external reuse

Extraction must preserve its public contract where feasible.
