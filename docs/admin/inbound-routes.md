# Inbound routes (DDI)

Route: `/inbound-routes`

1. Map inbound number (DDI) → destination (extension, queue, IVR, …), or attach a **route profile** for schedule-aware routing.
2. Save → **Commit**.

For how day timers, modes, and route profiles fit together, see [Day timers and route profiles](day-timers-and-profiles.md).

### Fleet DID ownership

Fleet catalog + SBC projection own **which instance** a DID routes to (hop 1). Instance inbound routes own **what happens** once the call lands (hop 2), including Class masks. Allocate does **not** auto-create inbound routes — see [DIDs — where they are allocated](../fleet/dids.md). Also [Tenant move](../fleet/tenant-move.md).

After SBC number-dialect normalize, the home always sees inbound DIDs as **+E.164** — prefer matching patterns accordingly ([Number dialects](../fleet/number-dialect.md), [DIDs](../fleet/dids.md)).
