# ADR-006: Scoped Authorization and Separation of Duties

- Status: Accepted
- Date: 2026-11-24

## Decision

The MVP uses RBAC plus mandatory resource scope (organization, programme, factory ownership/assignment and decision authority).

RBAC alone is not sufficient.

Sensitive programme-policy publication and selected governance actions support four-eyes approval where configured.

System administration does not automatically grant business decision visibility.

## Reason

Funding decisions require ownership-aware access, tenant/programme separation and protection against self-approved policy manipulation.
