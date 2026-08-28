# Site Groups (cross-tenant short dial)

**Product name:** Site Groups.  
**Tech / APIs / S3:** dial cohort (`dial-cohorts`, ability `fleet_dial_cohorts`).

Site Groups are the **supported** way for sister sites to short-dial each other on a fleet: each member owns a **destination routing prefix**; Gatekeeper projects a full mesh of dial-prefix rows onto each home. Phones dial `{prefix}{extension}` the same way everywhere in the group.

!!! tip "Release model"
    Hand-entered per-sender dial prefixes (instance **Outbound → Dial prefixes**) are **lab / break-glass only**. Do not invent production meshes that way. Use **Fleet → Site Groups**.

## Operator model

| Idea | Meaning |
|------|---------|
| **Routing prefix** | Digits that mean *this tenant* as a destination (default width **2**, allowed **2–4**; one width per Site Group) |
| **Site Group** | Explicit membership set; full mesh among members |
| **Isolate** | Tenant with no routing prefix / not in a group — no short-dial routes to or from the group |
| **Managed dial prefix** | Node-local row with `source=cohort` — read-only on the instance UI |

**Dial:** `{routing_prefix of peer}{their extension}`  
**Same digits on every member** for a given peer (no per-sender invent).

!!! note "Queues / IVRs / length"
    Short dial always appends exactly the **destination** tenant’s extension length (`ext_len`). Phone extensions of that length work as expected. A queue or IVR on the far site is reachable **only** if its dial number is the **same length** as that tenant’s `ext_len`. Longer or shorter service numbers stay local-only on that tenant — sister sites cannot short-dial them. Give any cross-site service number a dial code in the destination’s `ext_len` if you need Site Group reachability. (Engineering: `TENANT_SHORT_DIAL_REQUIREMENTS.md` §3.8.1.)

**Callback:** when the fleet CLIP path is live, the far end sees `{your routing_prefix}{your extension}` and can redial those digits through the same mesh.

## Where to click

1. Sign in to the SPA with a fleet session that includes **`fleet_dial_cohorts`** (or `fleet_admin`).
2. **Fleet → Site Groups** — list, create, open a group.
3. On the group: set **name** / **prefix width**, **Add tenant** with a **routing prefix**, **Sync now**, remove members, or decommission.
4. **Fleet → Tenants** shows each tenant’s routing prefix and Site Group link.

Instance **Outbound → Dial prefixes** still lists rows for visibility; managed (Site Group) rows are view-only. With the cohort feature on, inventing cross-tenant prefixes on the instance is forbidden (403 → use Site Groups).

## Typical ops

| Action | Effect |
|--------|--------|
| Create Site Group | Empty group; choose prefix width (2–4) |
| Add member + routing prefix | Catalog update + materialise job (mesh upsert + one Commit per home) |
| Change routing prefix | Rewrites peers’ digits for that destination |
| Remove member | Clears that tenant’s prefix/back-pointer; prunes managed rows |
| Sync now | Full reconcile; optional prune of unmanaged cross-tenant dialaliases on members |
| Decommission group | Soft-off; members become isolates |

Jobs are durable under the Gatekeeper catalog (retryable). Directory / Gatekeeper stay **off the call path** after projection.

## Lab interim (hand prefixes)

Before Site Groups, lab used **manual** dialalias rows: on calling tenant A, invent prefix → B’s FQDN; reverse row on B if needed. That path still exists for break-glass when the cohort feature flag is off.

| | Lab hand prefixes | Site Groups (ship) |
|--|-------------------|--------------------|
| Source of truth | Sender invents digits | Destination `routing_prefix` + membership |
| Reverse / mesh | Manual second row | Automatic project |
| Instance UI | Full CRUD | Managed rows read-only |
| Wild / release | **No** | **Yes** |

Converting **SARK InterSARK / INTERSITE** digit maps into interim prefixes (not into Site Groups yet) is a separate recipe: [Dial prefixes — InterSARK convert](dial-prefix-legacy-migrate.md). There is **no** product migrate from hand-invented wild meshes into Site Groups — that model is not shipping.

## Related

- Fleet control plane: [Fleet overview](overview.md)
- Spec (engineering): `pbx3/workingdocs/DIAL_COHORT_REQUIREMENTS.md`
