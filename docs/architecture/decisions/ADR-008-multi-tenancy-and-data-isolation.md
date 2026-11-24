# ADR-008: Multi-Tenancy and Strict Data Isolation

- Status: Accepted
- Date: 2026-11-24

## Context

The same deployed system must support multiple institutions while preventing one institution from reading, modifying, inferring, exporting, caching or accidentally joining another institution's data.

Examples may include industrial federations, implementing partners, donors, development agencies and government bodies.

Role-based access alone is insufficient because the same role may exist in more than one institution.

## Decision

Adopt:

**Shared application + shared PostgreSQL database + shared bounded-context schemas + tenant-keyed rows + PostgreSQL Row-Level Security (RLS).**

### Tenant

A `Tenant` is the primary security and data-isolation boundary.

It represents an isolated operational workspace.

### Organization

An `Organization` is a legal/institutional participant.

An organization may participate in one or more tenants through explicit memberships.

This separation allows a future joint programme workspace without opening one institution's private tenant data to another.

## Default Isolation Rule

Data is never shared across tenants by default.

Cross-tenant reads, joins, exports and references are prohibited.

If multiple institutions need to collaborate on one programme, the preferred MVP pattern is a dedicated joint tenant with explicit memberships.

Arbitrary cross-tenant sharing grants are out of scope for the MVP.

## Tenant Key

Every tenant-owned business record must include:

```text
tenant_id uuid not null
```

Tenant-local uniqueness must include `tenant_id`.

Where practical, relationships between tenant-owned tables use tenant-safe composite keys, for example:

```text
unique (tenant_id, application_id)

foreign key (tenant_id, application_id)
  references application.recovery_application(tenant_id, application_id)
```

This provides database-level protection against accidental cross-tenant references.

## PostgreSQL RLS

RLS is mandatory for tenant-owned tables in production-capable deployments.

Each transaction establishes validated tenant context using a transaction-local setting such as:

```sql
SET LOCAL app.current_tenant_id = '<validated-tenant-uuid>';
```

Tenant policies use:

```sql
USING (
  tenant_id = current_setting('app.current_tenant_id', true)::uuid
)
WITH CHECK (
  tenant_id = current_setting('app.current_tenant_id', true)::uuid
)
```

Use:

- `ENABLE ROW LEVEL SECURITY`
- `FORCE ROW LEVEL SECURITY`

The runtime role must not be superuser and must not have `BYPASSRLS`.

## Tenant Context Resolution

The client may request/select a tenant workspace, but the server validates that the authenticated actor has an active membership.

Never trust a raw client-supplied tenant ID.

Tenant context becomes part of the application request context.

## Connection Pool Safety

Because pooled connections are reused, tenant context must be transaction-local.

Persistent session-level tenant settings are prohibited.

## Defense in Depth

RLS is not the only tenant control.

Repositories and application services also require tenant context explicitly.

Authoritative operations validate:

- authenticated actor
- active tenant membership
- tenant status
- role
- programme/factory/resource scope
- separation-of-duties rules

## Background Jobs

Every job touching tenant data carries explicit `tenant_id`.

Workers re-establish and validate tenant context before data access.

Unscoped tenant jobs are prohibited.

## Cache / Queue / Object Storage / Search

Tenant isolation applies outside PostgreSQL.

Tenant-owned:

- cache keys
- queue messages
- object-storage paths
- exports
- temporary files
- offline cache
- search indexes

must include an isolation key or equivalent access boundary.

## Audit

Every tenant-scoped audit event includes `tenant_id`.

Future cross-tenant administrative access requires explicit break-glass authorization and high-severity audit events.

Routine system administration does not imply tenant business-data access.

## Global Control Plane

A minimal IAM/control-plane layer may be global:

- tenant registry
- organization registry
- global actor identity reference

All operational business data remains tenant-scoped.

## Export Rule

Exports are tenant-scoped by default.

Cross-tenant exports are prohibited in the MVP.

## Testing

Mandatory tenant-isolation tests include:

- Tenant A cannot read Tenant B rows
- Tenant A cannot insert rows with Tenant B tenant_id
- cross-tenant references fail
- background jobs cannot execute without tenant context
- cache/object-storage paths cannot collide across tenants
- export APIs cannot span tenants
- RLS remains enforced under normal runtime DB credentials

## Consequences

### Positive

- one system can safely serve multiple institutions
- defense-in-depth against tenant leakage
- supports institutional scale without code forks
- supports deliberately shared programme workspaces

### Negative

- most business tables require tenant-aware indexes/keys
- tests and migrations are more disciplined
- some analytics require explicit tenant-safe aggregation architecture

## Future Scale

A high-risk or very large tenant may later move to a dedicated database/deployment while preserving the same public contracts.
