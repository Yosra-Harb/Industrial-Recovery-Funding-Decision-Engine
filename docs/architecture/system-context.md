# System Context

## System Role

The Industrial Recovery Funding Decision Engine is a headless institutional decision-support component.

It is not intended to replace an organization's existing portal or enterprise system.

## External Actors

### Factory / Enterprise

Provides or updates recovery-related application data and supporting evidence.

### Federation / Implementing Organization

Owns or validates enterprise master data and manages operational workflow.

### Funding Provider

Defines programme budget, funding instrument, objectives, constraints, and portfolio strategy.

### Review Committee

Reviews recommendations and retains authority over final decisions.

### Auditor / Oversight

Reviews the trace of rules, configurations, recommendations, overrides, and outcomes.

### Existing Host Platform

Communicates with the engine through authenticated versioned APIs.

## High-Level Context

```text
                   Funding Provider
                         │
                         ▼
Existing Platform ──► Funding Programme
      │                  │
      │                  ▼
      ├──────────► Decision Engine ◄──────── Factory / Enterprise
      │                  │
      │                  ▼
      │          Recommended Portfolio
      │                  │
      ▼                  ▼
Operational UI      Human Committee
      │                  │
      └──────────┬───────┘
                 ▼
             Monitoring
                 │
                 ▼
              Outcomes
```

## Trust Boundaries

At minimum, distinguish:

1. client/browser boundary
2. API boundary
3. identity provider boundary
4. application/domain boundary
5. persistence boundary
6. optimization execution boundary
7. offline device boundary
8. external integration boundary

Detailed threat modeling will evolve before production pilot.
