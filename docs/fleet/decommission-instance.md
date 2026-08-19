# Decommission a fleet instance

Remove a **home box** (EC2 / node) from the fleet: tenants first, then catalog instance, SBC hygiene, then terminate AWS.

This is **not** the same as [Tenant delete](tenant-delete.md). Tenants and instances are different Fleet objects.

| Object | Fleet panel | Soft end-state |
|--------|-------------|----------------|
| **Tenant** (site / cluster) | **Tenants** | Hidden after Delete job (`status=decommissioned`) |
| **Instance** (Asterisk node) | **Instances** | Hidden after **Decom** (`status=decommissioned`) |

!!! tip "Rebuild instead?"
    Same KSUID + restore from S3 → [Rebuild a fleet node from S3](rebuild-from-s3.md).  
    New box with a **new** KSUID → finish this page, then [Commission a fleet instance](commission-instance.md).

## Before you start

Collect from Fleet → **Instances** (or catalog):

| Field | Example (lab) |
|-------|----------------|
| Label / Name | Toliman |
| Catalog **id** (KSUID) | `3HMsafJSDSYK90uCUoUbCsXRnHq` |
| Instance FQDN | `kildare.pbx3.com` |
| Public SIP IP | `3.93.253.1` |
| `sbc_dispatcher_setid` | `4` |
| Tenants still on this home | list under Fleet → **Tenants** filtered by home |

Also: SBC admin (`https://sbc.pbx3.com/admin` in lab), AWS console/CLI, DNS for the instance FQDN.

---

## Step 1 — Decide tenant fate

For **each** active tenant on this instance:

| Keep the site | Throw away the site |
|---------------|---------------------|
| [Tenant move](tenant-move.md) to another home **first** | [Tenant delete](tenant-delete.md) |

Do **not** terminate EC2 while active tenants still home here.

### DIDs (if deleting tenants)

1. Fleet → **DIDs** → **Release** each number on that tenant.  
   Release marks catalog `released` **and** updates SBC inbound routes in one step (no separate “re-project” click).
2. Then run Fleet → **Tenants** → **Delete** (type shortuid at confirm).  
   Soft-released DID history rows may remain grey in the DID list; that is audit residue, not live ownership.

Wait until each Delete job shows **completed** (green banner / state `completed`).

---

## Step 2 — Soft-decommission the instance (catalog)

Fleet → **Instances** → row menu → **Decom**.

- Hides the node from the instance picker (`status=decommissioned`).
- Does **not** stop Asterisk, terminate EC2, or delete S3 backups.
- Ability: same fleet manage path as other instance lifecycle actions.

After EC2 is gone (or catalog row no longer needed), **Remove** on a **decommissioned** row drops the catalog entry. S3 `instances/{KSUID}/meta.json` and backups are kept.

**CLI equivalent** (Mac / ops):

```bash
cd pbx3/pbx3-directory/tools
export PBX3_ORG_BUCKET=08jzwn-pbx3   # your org bucket
./unregister-instance.sh --id {KSUID} --notes 'Replacing node'
# Drop the catalog row entirely (meta kept under instances/{id}/):
# Fleet → Instances → Remove (decommissioned row), or:
# ./unregister-instance.sh --id {KSUID} --remove --notes 'EC2 terminated'
```

After **Decom**, the instance **must not** appear as Active. Tenants for that home should already be gone from **Tenants**.

---

## Step 3 — SBC hygiene

Tenant Delete already removed the tenant **SIP domain**. Instance **Decom** does **not** tear down edge routing for the home.

Open the SBC admin UI (lab: `https://sbc.pbx3.com/admin`).

### 3a — Peers

**Peers** → remove the Asterisk / home Peer for this node (if present).

### 3b — Fail2ban whitelist

**Fail2ban → Whitelist** → remove this home’s public IP `/32` (lab gap until fleet-home auto-whitelist ships).

### 3c — Dispatcher destinations (setid)

There is **no** top-level **Dispatcher** menu. Destinations live under **Domain Routes → Manage destinations**, and for **fleet-locked** setids the SBC admin **refuses** create/edit/delete (catalog / Provision edge is the author — Rule 13).

| Situation | What to do |
|-----------|------------|
| No domains left on that setid; orphan `fleet=node` destination | **Usually leave it.** Harmless for call routing. Next greenfield home gets a **new** setid via Fleet → Instances → **Provision edge**. |
| Must remove now | Break-glass only: SSH / SQL / OpenSIPS MI on the SBC, then keep catalog honest. Not a sticky Filament path. |

There is **no** Fleet “unprovision edge” action yet.

---

## Step 4 — DNS

Remove or park the instance **A** record (e.g. `kildare.pbx3.com`).  
Fleet nodes do **not** publish public tenant A records.

---

## Step 5 — Terminate the EC2

1. AWS → stop/terminate the instance.
2. Release the **EIP** only if you will not reuse it on the replacement.
3. Optional later: detach/delete the node-scoped IAM instance profile / role.

!!! warning "Order matters for noise"
    Prefer **Decom → SBC Peers/WL → terminate**.  
    Terminating while the catalog row is still **active** causes Gatekeeper `/up` probe failures and ops mail until Decom.

---

## Step 6 — Replacement (optional)

| Goal | Path |
|------|------|
| New identity (new KSUID / opaque `{shortuid}.apex`) | [Commission a fleet instance](commission-instance.md) |
| Same KSUID + latest S3 backup | [Rebuild from S3](rebuild-from-s3.md) |

Then Fleet Create / move tenants back, re-allocate DIDs, smoke in/out calls, Fleet → Catalog reconcile clean.

---

## Checklist

- [ ] Tenants moved or Delete jobs `completed`
- [ ] DIDs released (if not keeping ownership)
- [ ] Instance **Decom** (or `unregister-instance.sh`)
- [ ] Instance **Remove** from catalog when retired (decommissioned row, or `unregister-instance.sh --remove`)
- [ ] SBC Peer removed
- [ ] Fail2ban whitelist IP removed
- [ ] DNS instance A removed/parked
- [ ] EC2 terminated (EIP decision made)
- [ ] (If replacing) [commission](commission-instance.md) + tenants/DIDs restored

## Related

- [Tenant delete](tenant-delete.md) — per-site wipe + soft catalog meta  
- [Commission a fleet instance](commission-instance.md) — new home (install → edge)  
- [Onboard a second instance](onboard-instance.md) — adopt-only for a healthy node  
- [Rebuild from S3](rebuild-from-s3.md) — same KSUID recovery  
- Product locks: `FLEET_TENANT_DELETE_REQUIREMENTS.md`, `FLEET_DOMAIN_SETID_LOCK.md` (in `pbx3` / `pbx3-directory`)
