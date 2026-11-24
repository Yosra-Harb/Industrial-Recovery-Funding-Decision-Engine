# Database Schema v0.2 — Multi-Tenant Security Baseline

## Status

This version **supersedes Database Schema v0.1 for tenancy and security rules**.

Database: PostgreSQL  
Deployment model: shared application + shared database + bounded-context schemas + strict tenant isolation.

The detailed business fields defined in v0.1 remain valid unless changed here.

---

# 1. Isolation Model

## Tenant

`tenant_id` is the primary business-data isolation key.

Every tenant-owned table includes:

```text
tenant_id uuid not null
```

No tenant-owned business row exists without tenant ownership.

## Organization

An Organization is a legal/institutional entity.

A Tenant is an isolated workspace.

An Organization may participate in multiple Tenants through explicit membership.

A joint programme between organizations should use a dedicated Tenant rather than cross-tenant sharing.

---

# 2. IAM / Control Plane

Control-plane tables are global but accessible only through Identity & Access.

## iam.organization

| Field | Type | Null |
|---|---|---:|
| organization_id | uuid PK | no |
| organization_name | varchar(255) | no |
| organization_type | varchar(64) | no |
| status | varchar(32) | no |
| external_reference | varchar(255) | yes |
| created_at | timestamptz | no |
| updated_at | timestamptz | no |
| row_version | bigint | no |

## iam.tenant

| Field | Type | Null |
|---|---|---:|
| tenant_id | uuid PK | no |
| tenant_name | varchar(255) | no |
| tenant_type | varchar(64) | no |
| status | varchar(32) | no |
| created_at | timestamptz | no |
| updated_at | timestamptz | no |
| row_version | bigint | no |

Possible tenant types:

- INSTITUTION
- JOINT_PROGRAMME
- SANDBOX
- PILOT

## iam.tenant_organization_membership

| Field | Type | Null |
|---|---|---:|
| tenant_id | uuid FK → iam.tenant | no |
| organization_id | uuid FK → iam.organization | no |
| relationship_code | varchar(64) | no |
| valid_from | timestamptz | no |
| valid_to | timestamptz | yes |
| created_at | timestamptz | no |

Primary key:

```text
(tenant_id, organization_id)
```

## iam.actor_reference

Revised fields:

| Field | Type | Null |
|---|---|---:|
| actor_id | uuid PK | no |
| external_subject | varchar(255) | no |
| actor_type | varchar(32) | no |
| status | varchar(32) | no |
| created_at | timestamptz | no |
| updated_at | timestamptz | no |
| row_version | bigint | no |

An actor is global identity metadata only.

Tenant access is granted through membership/authorization rows.

## iam.actor_tenant_membership

| Field | Type | Null |
|---|---|---:|
| membership_id | uuid PK | no |
| actor_id | uuid FK → iam.actor_reference | no |
| tenant_id | uuid FK → iam.tenant | no |
| organization_id | uuid FK → iam.organization | yes |
| status | varchar(32) | no |
| valid_from | timestamptz | no |
| valid_to | timestamptz | yes |
| created_at | timestamptz | no |

Constraint:

```text
unique(actor_id, tenant_id, organization_id)
```

## iam.authorization_assignment

Tenant becomes mandatory scope.

| Field | Type | Null |
|---|---|---:|
| assignment_id | uuid PK | no |
| actor_id | uuid FK → iam.actor_reference | no |
| tenant_id | uuid FK → iam.tenant | no |
| role_code | varchar(64) | no |
| organization_scope | uuid | yes |
| programme_scope | uuid | yes |
| factory_scope | uuid | yes |
| valid_from | timestamptz | no |
| valid_to | timestamptz | yes |
| created_at | timestamptz | no |

Authorization is evaluated as:

```text
authenticated actor
+ active tenant membership
+ tenant status
+ role
+ resource scope
+ separation-of-duties rules
```

---

# 3. Tenant-Owned Business Tables

The following schemas are tenant-owned:

- `factory`
- `programme`
- `application`
- `evaluation`
- `optimization`
- `decision`
- `monitoring`
- tenant-scoped `audit`

Every table in these schemas receives `tenant_id uuid not null`.

Examples:

```text
factory.factory
  tenant_id
  factory_id
  ...

programme.funding_programme
  tenant_id
  programme_id
  ...

application.recovery_application
  tenant_id
  application_id
  ...

evaluation.evaluation_run
  tenant_id
  evaluation_run_id
  ...

optimization.optimization_run
  tenant_id
  optimization_run_id
  ...

decision.human_decision
  tenant_id
  decision_id
  ...

monitoring.monitoring_outcome
  tenant_id
  outcome_id
  ...

audit.audit_event
  tenant_id
  audit_event_id
  ...
```

---

# 4. Tenant-Safe Keys

Primary UUIDs may remain globally unique, but database relationships must also verify tenant equality.

For a tenant-owned parent table:

```sql
UNIQUE (tenant_id, application_id)
```

For a child table:

```sql
FOREIGN KEY (tenant_id, application_id)
REFERENCES application.recovery_application (tenant_id, application_id)
```

This pattern applies to same-context tenant-owned relationships.

It prevents accidental Tenant A → Tenant B references.

Cross-context database foreign keys remain prohibited by default under ADR-007.

---

# 5. Tenant-Aware Uniqueness

Never use a global unique constraint for a tenant-local business value unless it is intentionally global.

Prefer:

```text
unique (tenant_id, external_reference)
unique (tenant_id, code, version_no)
unique (tenant_id, programme_id, version_no)
```

rather than:

```text
unique (external_reference)
unique (code, version_no)
```

Global identity records such as OIDC subject may remain globally unique.

---

# 6. PostgreSQL Row-Level Security

RLS is mandatory on tenant-owned tables.

Generic policy pattern:

```sql
ALTER TABLE application.recovery_application ENABLE ROW LEVEL SECURITY;
ALTER TABLE application.recovery_application FORCE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation_recovery_application
ON application.recovery_application
USING (
  tenant_id = current_setting('app.current_tenant_id', true)::uuid
)
WITH CHECK (
  tenant_id = current_setting('app.current_tenant_id', true)::uuid
);
```

The same pattern is applied to each tenant-owned table with deployment-tested policies.

## Missing Tenant Context

No tenant-owned query should succeed when tenant context is absent.

The application must translate missing/invalid tenant context into a fail-closed authorization error.

---

# 7. Runtime Database Roles

Separate responsibilities:

### migration_admin

- migrations only
- not used by application runtime

### app_runtime

- no superuser
- no BYPASSRLS
- context-scoped schema privileges
- no DDL

### audit_writer

- append-only audit insert capability
- no audit UPDATE/DELETE

### background_worker

- no BYPASSRLS
- tenant context required per job
- minimum required schema privileges

Future extraction may use per-context roles.

---

# 8. Request Transaction Pattern

Authoritative request flow:

```text
Authenticate
↓
Resolve actor
↓
Resolve requested tenant
↓
Validate active actor_tenant_membership
↓
Resolve role/resource scopes
↓
Begin DB transaction
↓
SET LOCAL app.current_tenant_id = validated tenant
↓
Execute repository/application operation
↓
Audit event
↓
Commit
```

The raw tenant ID from a browser is never trusted by itself.

---

# 9. Connection Pool Rule

Tenant context must use transaction-local settings.

Forbidden:

```sql
SET app.current_tenant_id = '...';
```

across pooled sessions.

Required pattern:

```sql
BEGIN;
SET LOCAL app.current_tenant_id = '...';
...
COMMIT;
```

This prevents tenant context leaking between requests that reuse a pooled connection.

---

# 10. Background Jobs

Every tenant job envelope includes:

- job_id
- tenant_id
- actor/service context
- operation type
- resource reference
- idempotency key
- schema version
- created_at

Worker flow:

```text
receive job
→ validate tenant
→ validate service/job authorization
→ start transaction
→ SET LOCAL tenant context
→ execute
→ audit
```

A tenant-data job without tenant_id is rejected.

---

# 11. Cache / Queue / Object Storage

## Cache

Keys use tenant namespace:

```text
tenant:{tenant_id}:...
```

No unqualified business cache key is allowed.

## Queue

Tenant ID is mandatory in message envelope.

## Object Storage

Evidence paths use tenant isolation:

```text
tenant/{tenant_id}/application/{application_id}/evidence/{evidence_id}
```

Storage access is mediated by the server or short-lived scoped signed URLs.

Do not expose permanent public object URLs.

## Exports

Exports are generated inside one tenant context.

Cross-tenant export is prohibited in MVP.

---

# 12. Offline Storage

Offline records include tenant ID, but tenant ID alone does not authorize them.

Offline cache is:

- minimal
- encrypted
- tenant-scoped
- user/session scoped where practical
- expiry controlled

On reconnect, the server revalidates:

- tenant membership
- resource authorization
- base row version
- operation idempotency

before applying queued writes.

---

# 13. Audit Tenant Isolation

Tenant-scoped events include:

```text
tenant_id
actor_reference
event_type
entity_type
entity_id
correlation_id
event_hash
previous_event_hash
occurred_at
```

Routine tenant users cannot query another tenant's audit records.

Cross-tenant platform-security events belong to a separately controlled system-security audit stream.

---

# 14. Indexing Rule

Most tenant-owned lookup indexes start with `tenant_id`.

Examples:

```sql
CREATE INDEX ...
ON application.recovery_application
(tenant_id, programme_id, submission_status);

CREATE INDEX ...
ON evaluation.evaluation_run
(tenant_id, policy_version_id, status, started_at);

CREATE INDEX ...
ON decision.budget_commitment
(tenant_id, programme_id, status);
```

This supports both security filtering and query performance.

---

# 15. Security Tests Required Before Migration Approval

The executable migration is not accepted until automated tests demonstrate:

1. Tenant A cannot SELECT Tenant B data.
2. Tenant A cannot INSERT/UPDATE a row as Tenant B.
3. tenant-safe foreign keys reject cross-tenant relationships.
4. missing tenant context fails closed.
5. runtime role cannot bypass RLS.
6. pooled connections do not leak tenant context.
7. background jobs fail without explicit tenant context.
8. exports remain tenant-scoped.
9. evidence storage access remains tenant-scoped.
10. audit queries respect tenant boundaries.

---

# 16. Current Remaining Decisions

Before executable migration:

1. UUIDv7 vs UUIDv4
2. stable status-code catalogue
3. canonical hashing/serialization
4. object-storage abstraction
5. BudgetCommitment locking strategy
6. retention policy by data classification

The multi-tenancy model is now resolved by ADR-008.
