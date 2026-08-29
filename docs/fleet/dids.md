# DIDs — where they are allocated

On a **fleet**, bringing a carrier number into service is **two hops**. Do not conflate them.

| Hop | Question | Where you author it |
|-----|----------|---------------------|
| **1 — Delivery** | Which **tenant home** (instance) receives the INVITE from the carrier? | **Fleet → DIDs** (catalog → project onto the SBC) |
| **2 — Behaviour** | Which **extension / IVR / queue** (or Class mask) handles the call once it lands? | **Instance → Inbound routes** |

Solo / non-fleet installs have no Fleet DIDs panel: hop-1 may be authored on the SBC Number routes UI; hop-2 is still instance inbound routes.

## Hop 1 — Fleet DIDs (allocate here)

**Panel:** SPA **Fleet mode → DIDs**.

1. **Allocate** (or reassign) an E.164 to a tenant shortuid.
2. Choose **singleton** or **block** delivery when the catalog supports it (block uses a digit E.164 `sip_prefix` after dialect normalize — not a UK national leading-`0` form).
3. Catalog writes `tenants/{shortuid}/dids.json` (home of record for ownership).
4. **Project → SBC** compiles inbound delivery rules on the edge (`fleet=did` Number routes → that tenant’s current home).

**Status (catalog):**

| Status | Meaning |
|--------|---------|
| `active` / `porting` | Deliverable — projected to the SBC |
| `reserved` | Owned in catalog; not yet expected to ring (no requirement that hop-2 exists yet) |
| `released` | Soft-removed from delivery; may still show in the list until hard-remove |

**Number shape:** after SBC inbound dialect normalize, hop-1 match digits are **digit E.164** (e.g. `441924918076`). Prefer allocating in that form. See [Number dialects](number-dialect.md).

### What Allocate does *not* do

Fleet Allocate does **not** create the instance inbound-route row. That looks like two gestures for one DID; it is intentional.

Hop-2 supports **Class** and other masks (many consecutive DIDs → one destination). Auto-creating one inbound route per Allocate would fight that pattern. Keep hop-2 on the instance.

## Hop 2 — Instance inbound routes

**Panel:** instance SPA **Inbound routes** (see [Inbound routes (DDI)](../admin/inbound-routes.md)).

- Create or edit **DiD**, **CLiD**, or **Class** rows.
- Prefer **`+E.164`** patterns for DiD match after dialect normalize (e.g. `+441924918076`).
- Set open / closed destinations (or a route profile), Save → **Commit**.

Typical sequence for a new singleton:

1. Fleet → DIDs → Allocate (+ Project).
2. Instance → Inbound routes → Create (or a Class that already covers the number).

## SBC Number routes (edge)

The SBC **Number routes** table is the **compiled** edge for drouting — not the fleet authorship surface for PBX3 tenant DIDs.

| Direction | Fleet-joined SBC | Standalone / general SBC |
|-----------|------------------|---------------------------|
| **Outbound** (group 0) | Author here — dialled prefix → carrier Peer | Author here |
| **Inbound** (group 1) | **Do not author here** — use Fleet → DIDs. Filament hides inbound create/list when the SBC is fleet-joined | May author hop-1 here |

Fleet-projected inbound rows carry `fleet=did`. Retarget ownership or home via **Fleet → DIDs** (or tenant move + re-project), not by editing those rows on the SBC.

## Tenant move

When a tenant moves home:

- Hop-2 (`inroutes`) travels with the tenant database.
- Hop-1 must follow the new home (catalog `instance_id` + SBC delivery re-project). Domain / phone cutover is separate from DID delivery; operators should confirm inbound after move (Project / reconcile if delivery lags).

See [Tenant move](tenant-move.md).

## Quick reference

| I want to… | Use |
|------------|-----|
| Assign a Magrathea Telecom / Gamma / … number to a tenant | **Fleet → DIDs → Allocate** |
| Send a range of DIDs to one IVR / queue | **Instance → Inbound routes** (Class / mask) after hop-1 delivery exists |
| Change which instance receives a DID | Fleet reassign / move + Project (not SBC inbound edit) |
| Choose outbound carrier by dialled prefix | **SBC → Number routes** (outbound) |
| Fix wire format for a carrier Peer | **SBC → Peers** number dialect — [Number dialects](number-dialect.md) |

## Related

- [Fleet overview](overview.md)
- [Inbound routes (DDI)](../admin/inbound-routes.md)
- [Number dialects](number-dialect.md)
- [Tenant move](tenant-move.md)
- [Install SBC edge](install-sbc.md)
