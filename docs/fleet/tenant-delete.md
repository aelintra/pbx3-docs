# Tenant delete (Fleet)

Fleet owns delete on fleet-joined nodes (**Rule 14** durable job — not a single browser POST). Spec: `pbx3/workingdocs/FLEET_TENANT_DELETE_REQUIREMENTS.md`.

## Operator path

1. Fleet → **Tenants** → **Delete** on the row (ability `fleet_instances`).
2. Job opens at **awaiting_confirm**. Type the tenant **shortuid** and confirm.
3. Automated stages:
   - **removing_edge** — SBC `DELETE /fleet/domains/{fqdn}` (stops new REGISTER)
   - **wiping_node** — node `DELETE /fleet/tenants/{shortuid}` + cert sync / Commit (best-effort)
   - **catalog** — soft-decommission (`status=decommissioned`; hidden from Tenants list)
4. Reopen anytime via Fleet → **Jobs** → Delete jobs → **Open**. You do not need to stay on the page.

**Abort** is safe **before** node wipe. If the SBC domain was already removed, use **Register on SBC** to repair.

## Notes

- Solo / non-fleet instances still use on-node Sanctum Delete.
- Catalog **soft-decommission** keeps `meta.json` for audit; S3 backups / recordings are **not** auto-purged. Soft-delete **is** the catalog end-state in v1 (no separate “finalize / hard purge” yet).
- **Media trees** on the node are a known wipe gap (v1).
- DIDs still attached are warned in preflight — **Release** via Fleet DIDs first (release updates SBC inbound routes in the same action; no extra project step).
- Removing the **home EC2** after tenants are gone: [Decommission a fleet instance](decommission-instance.md).
- **FQDN rename** is cancelled (naming lock); not part of Delete.

## Lab check

1. Fleet Create a throwaway tenant → Delete → confirm shortuid.  
2. SBC domain gone; node cluster gone; catalog status `decommissioned`.  
3. Job state `completed`.
