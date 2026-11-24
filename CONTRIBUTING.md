# Contributing

## Core Rule

Every contribution must preserve **module independence and extensibility**.

## Before Coding

A change must identify:

1. owning module
2. input contract
3. output contract
4. persistence impact
5. security/privacy impact
6. offline impact
7. tests required
8. whether an ADR is required

## Module Boundaries

Forbidden by default:

- direct access to another module's private tables
- imports from another module's infrastructure internals
- hard-coded funding programme rules
- UI-only enforcement of business/security rules
- circular dependencies
- shared global mutable state
- hidden cross-module side effects

Preferred:

- explicit interfaces
- application services
- domain events when justified
- versioned DTO/contracts
- dependency inversion
- configuration-driven behaviour

## Testing Requirement

Changes must include appropriate tests:

- unit tests for domain logic
- contract tests for module interfaces
- integration tests for persistence/infrastructure
- regression tests for defects
- end-to-end tests for critical workflows

## Commit Style

Prefer Conventional Commits:

```text
feat:
fix:
docs:
test:
refactor:
chore:
security:
```

Example:

```text
feat(eligibility): add configurable sector rule evaluation
```

## Architecture Decision Records

Create an ADR when a change:

- changes module boundaries
- introduces a major dependency
- changes persistence ownership
- changes API compatibility
- introduces a new infrastructure pattern
- weakens an existing architectural constraint

## Definition of Done

A feature is not complete until:

- code is tested
- contracts are documented
- security/privacy reviewed
- audit requirements considered
- offline behaviour considered
- documentation updated
- no unrelated module is broken
