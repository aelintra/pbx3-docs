# Tenants

Routes: `/tenants`, `/tenants/new`, `/tenants/:pkey`

## Create

1. **Tenants** → New.
2. Fill required fields → Save.
3. FQDN is assigned as `{shortuid}.{domain}` and is **read-only** afterward.
4. Add DNS **A** for the tenant FQDN → this node.
5. **Certificates** → [Sync with tenant list](../tls/sync-tenants.md).
6. Top bar → **Commit**.

## Edit

Open tenant → change allowed fields → Save → **Commit**.

## Fleet note

Moving a tenant between instances is a fleet operation — see [Tenant move](../fleet/tenant-move.md). Do not invent a second FQDN for the same tenant on another node without a move.
