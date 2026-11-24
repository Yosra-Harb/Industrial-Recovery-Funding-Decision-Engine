# ADR-005: Offline Authority Boundary

- Status: Accepted
- Date: 2026-11-24

## Decision

Offline operation supports field continuity but not independent funding authority.

Offline MVP may capture/edit drafts, cache minimum reference data, queue evidence operations and perform advisory local validation.

Programme publication, official evaluation finalization, official optimization, final decisions, overrides and budget commitments require online server authorization.

Offline writes use idempotency keys and base-version conflict detection.

## Reason

This balances unstable-connectivity support with security, policy freshness, budget integrity and auditability.
