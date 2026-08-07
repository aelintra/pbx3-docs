# Sync certificate (Let's Encrypt)

!!! danger "DO NOT create DNS A records for tenant domains (SBC fleet)"
    Tenant FQDNs are **not** published in DNS. Sync issues the **instance** hostname only. If you added tenant A records “to make LE work,” remove them — that path is wrong for fleet.

**Renew** keeps the **old** SAN list on disk. Use **Sync certificate** to re-issue with the **intended** list.

## SBC fleet (default)

Lock: node LE = **instance FQDN only** — no tenant public A records, no tenant SANs.

1. Instance A record → this node.
2. SPA → **Certificates** → **Sync certificate**.
3. **Cert covers:** should list **only** the instance hostname.

CLI: `sudo /opt/pbx3/scripts/le-sync-cert-sans.sh you@example.com instance.example.com`

## Solo / direct-to-node

Option A multi-SAN (node + tenants that resolve here):

1. Each tenant FQDN has DNS **A → this node**.
2. SPA → **Certificates** → **Sync certificate**.
3. Confirm **Cert covers:** lists node + tenants.

## Tenant move

- **Fleet:** OpenSIPS setid + catalog — **not** tenant DNS/SAN.
- **Solo/direct:** dest Sync after import; source Sync after wipe.

**Do not** rely on Renew alone to grow or shrink SANs.
