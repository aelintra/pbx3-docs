# Sync when tenants change

Let's Encrypt **Renew** keeps the **old** SAN list. After you add/remove tenants, restore, or move tenants:

1. Ensure each tenant FQDN has DNS **A → this node**.
2. SPA → **Certificates** → **Sync with tenant list**.
3. Confirm **Cert covers:** lists node + tenants.

Use packages that include SAN replace (`le-sync-cert-sans.sh`, **≥ 0.0.3-17** in the lab line).

## Tenant move (certs)

1. Destination: Sync after import (add SAN).
2. DNS cutover.
3. Source: delete tenant → Sync (drop SAN).

**Do not** rely on Renew alone to grow or shrink SANs.
