# Ingest a compatible tenant database

Bring an **external**, already-built tenant sqlite onto a **fleet home**, enroll it in the catalog and on the SBC, then attach carrier numbers if needed.

This is **not** Fleet → Tenants → **Create** (empty tenant). It is for a **compatible database**: one sqlite file that already contains a single tenant’s configuration (extensions, inbound routes, outbound routes, CoS, and so on).

**Who runs this:** fleet / ops (MSP / NOC). There is no end-customer upload UI in v1.

**Related:** [DIDs — where they are allocated](dids.md) · [Tenant move](tenant-move.md) · [Tenant delete](tenant-delete.md)

---

## What a compatible database is

| Requirement | Detail |
|-------------|--------|
| Format | One sqlite file (`.db`) readable on the home |
| Tenants | **Exactly one** `cluster` row |
| Name | `cluster.pkey` is the human **Name** — must be non-empty, **not** `default`, and free on the home. Ingest does **not** invent or remint Name. See Step 0 to change the Name from 'default' to something else.  'default' is not allowed as a new Name. |
| Identity | `cluster.shortuid` and `cluster.id` are opaque; preserved if free, reminted on collision |
| Instance tables | May include `globals` / `trunks` for offline inspection — the home **ignores** those on ingest and keeps its own |

Unsuitable: multi-tenant combined DBs, empty create-only shells, or mobility **move** zips (use [Tenant move](tenant-move.md) for those).

**Single-tenant / solo sites** often ship with `cluster.pkey = default`. That fails preflight — rename **before** ingest (next section).

---

## End-to-end sequence

```text
Compatible .db
  → 0. Fix Name if pkey is default / empty / colliding
  → 1. Ingest on home (CLI)
  → 2. Catalog meta (+ label = Name)
  → 3. SBC domain
  → 4. Fleet DID Allocate (hop-1) if PSTN needed
  → 5. Hop-2 / Commit / smoke
```

Do **not** stop after step 1 — you will end up with a **node-only** tenant (catalog and SBC domain missing). Do **not** treat “domain registered” as finished for PSTN: steps 0–3 never route DIDs on the SBC — that is step 4 (Fleet DID Allocate + Project).

---

## 0. Set Name on the candidate DB (if needed)

**Name is established only in the candidate sqlite** (`cluster.pkey`). The CLI has no `--name` / rename flag.

Check:

```bash
sqlite3 /path/to/tenant.db "SELECT pkey, shortuid, id FROM cluster;"
```

If `pkey` is `default`, empty, or already used on the target home, edit a **copy** of the file (keep the original):

```bash
cp /path/to/tenant.db /path/to/tenant-renamed.db
sqlite3 /path/to/tenant-renamed.db \
  "UPDATE cluster SET pkey = 'AcmeOffice' WHERE pkey = 'default' OR pkey = '' OR pkey IS NULL;"
sqlite3 /path/to/tenant-renamed.db "SELECT pkey, shortuid, id FROM cluster;"
```

Use a short human Name that is free on the home (same rules as Fleet Create). Then ingest the renamed copy.

---

## 1. Ingest on the home (CLI)

Copy the compatible `.db` (after any Name fix) to the target fleet instance (SSH), then:

```bash
# Plan only — Name, shortuid, FQDN, row counts, collision remints
sudo -u www-data php /opt/pbx3api/artisan tenant:ingest-built /path/to/tenant.db --dry-run

# Apply merge into the home sqlite
sudo -u www-data php /opt/pbx3api/artisan tenant:ingest-built /path/to/tenant.db
```

What apply does:

- Merges that tenant’s cluster-scoped rows into the home database  
- Strips instance-owned tables from the file (`globals`, `trunks`, …)  
- Sets FQDN to `{shortuid}.{apex}` from the home’s domain  
- On a fleet home, rewrites that tenant’s outbound route paths to **Egress**  
- Does **not** create catalog meta, SBC domain, or DID delivery  

Note the printed **shortuid**, **FQDN**, and **Name** (`pkey`). If shortuid or id collided, the CLI remints them and prints old → new.

---

## 2. Register in the catalog

From an ops Mac (org-bucket IAM), write tenant meta so Fleet lists the tenant and homes it on this instance:

```bash
cd /path/to/pbx3-directory/tools
export PBX3_ORG_BUCKET=…          # e.g. 08jzwn-pbx3
export AWS_DEFAULT_REGION=us-east-1

./register-tenant.sh \
  --tenant-shortuid <shortuid> \
  --instance-id <home globals.id> \
  --cname <shortuid>.<apex> \
  --fqdn <shortuid>.<apex>
```

Home instance id is `globals.id` on the node (or the Fleet Instances row id).

This script sets shortuid / home / FQDN only — **not** Name. Authoritative Name is already on the home as `cluster.pkey` from step 1. For Fleet display, catalog meta **`label`** should equal that `pkey` (Create path sets this; patch `tenants/{shortuid}/meta.json` if `label` is missing).

---

## 3. Register the SIP domain on the SBC

The edge must know `{shortuid}.{apex}` → this home’s dispatcher setid.

- SPA **Fleet mode → Tenants** → **Register on SBC** for that tenant, **or**  
- Equivalent Gatekeeper domain enroll for the home’s `sbc_dispatcher_setid`

Until this succeeds, phones cannot REGISTER to the new FQDN via the SBC.

---

## 4. Attach DIDs (hop-1) — if PSTN is required

Ingest and domain enroll **do not** allocate carrier numbers. Delivery is always authored in Fleet DIDs.

**Panel:** SPA **Fleet mode → DIDs** — see [DIDs — where they are allocated](dids.md).

1. **Allocate** (or re-allocate) each number or **block** to this tenant’s **shortuid**.  
2. Confirm **Project → SBC** so inbound `fleet=did` rules point at this home’s setid.  
3. Prefer digit / `+E.164` shapes consistent with your dialect.

**Skip** this stage only if the tenant is intentionally non-PSTN (extensions / site dial only).

### What Allocate does *not* do

Hop-1 does **not** create or rewrite instance **Inbound routes** (hop-2). A compatible DB may already contain DiD / Class / CLiD rows; those stay on the home. If hop-2 is missing or wrong for the new wire face, fix it on the instance after Allocate.

---

## 5. Commit and verify

1. On the home SPA: review routes / inbound / extensions as needed.  
2. **Commit** so Asterisk regenerates (extensions, PJSIP, parks, CoS, …).  
3. Smoke:

   | Check | Expect |
   |-------|--------|
   | Softphone / desk REGISTER | To `{shortuid}.{apex}` via SBC |
   | Internal dial | Extensions within the tenant |
   | Outbound | Via **Egress** / SBC |
   | Inbound DID (if allocated) | Carrier → SBC hop-1 → home → hop-2 destination |

---

## Partial failure

| Failed after | What to do |
|--------------|------------|
| Ingest CLI error | Nothing durable; fix Name (`default` / empty / collision) or DB shape and retry |
| Merge OK, catalog missing | Re-run `register-tenant.sh` with the printed shortuid/FQDN |
| Catalog OK, no SBC domain | **Register on SBC** (repair) |
| Domain OK, no PSTN | Fleet → DIDs → Allocate + Project |
| DID OK, no ring to ext | Fix hop-2 inbound on the instance + Commit |

Do not auto-wipe a successful merge to “retry from scratch” without an intentional [Tenant delete](tenant-delete.md).

---

## Versus Create and Move

| Path | Use when |
|------|----------|
| **Create** | Brand-new empty tenant on a home |
| **Ingest compatible DB** (this page) | External built tenant DB → first home on **this** fleet |
| **[Tenant move](tenant-move.md)** | Tenant already on this fleet; relocate between homes (mobility zip) |

---

## Operator checklist

- [ ] Compatible `.db` (one `cluster`)  
- [ ] Name set: `pkey` not `default`/empty; free on the home (step 0)  
- [ ] `tenant:ingest-built --dry-run` then apply  
- [ ] Catalog meta; `label` = `pkey` (patch if script omitted it)  
- [ ] SBC domain registered  
- [ ] Fleet DID Allocate + Project (if PSTN)  
- [ ] Hop-2 inbound OK (`+E.164` where required)  
- [ ] Commit + REGISTER / dial / DID smoke  

