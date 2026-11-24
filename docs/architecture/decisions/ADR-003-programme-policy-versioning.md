# ADR-003: Immutable Programme Policy Versions

- Status: Accepted
- Date: 2026-11-24

## Decision

FundingProvider, FundingProgramme identity, PortfolioStrategy and related policy definitions belong to Programme Catalogue & Policy.

Every published programme policy is immutable.

Changing eligibility, constraints, weights, evidence requirements, risk settings or funding limits creates a new ProgrammePolicyVersion.

Policy publication records author, approver, timestamp and content hash.

## Reason

Historical funding decisions must remain reproducible and auditable even after programme rules change.
