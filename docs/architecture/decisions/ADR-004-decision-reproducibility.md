# ADR-004: Decision Reproducibility and Solver-Neutral Optimization

- Status: Accepted
- Date: 2026-11-24

## Decision

Official decision runs must reference immutable input snapshots and enough execution metadata to reconstruct or explain the historical result.

Optimization is defined by a solver-neutral PortfolioProblem contract.

Official runs record adapter/solver versions, parameters, seed, thread count when relevant, time/gap limits, input hash and candidate-set version/hash.

## Reason

A result that cannot be reconstructed after rules, data or solver versions change is not sufficiently auditable for institutional funding decisions.
