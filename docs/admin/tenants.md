# Tenants

Routes: `/tenants`, `/tenants/new`, `/tenants/:pkey`

## Create

1. **Tenants** → New.
2. Fill required fields → Save.
3. FQDN is assigned as `{shortuid}.{domain}` and is **read-only** afterward (SIP domain identity).
4. **SBC fleet:** no public tenant A record — skip DNS. **Solo/direct:** Add DNS **A** for the tenant FQDN → this node, then **Certificates** → [Sync certificate](../tls/sync-tenants.md).
5. Top bar → **Commit**.

## Edit

Open tenant → change allowed fields → Save → **Commit**.

## Fleet note

Moving a tenant between instances is a fleet operation — see [Tenant move](../fleet/tenant-move.md). Cutover is SBC `domain` setid + catalog, not tenant DNS.
