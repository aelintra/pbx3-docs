# Inbound routes (DDI)

Route: `/inbound-routes`

1. Map inbound number (DDI) → destination (extension, queue, IVR, …).
2. Save → **Commit**.

### Fleet DID ownership

Fleet catalog + SBC projection own **which instance** a DID routes to. Instance inbound routes own **what happens** once the call lands. See Fleet **DIDs** panel and [Tenant move](../fleet/tenant-move.md).

After SBC number-dialect normalize, inbound R-URIs are **+E.164** — prefer matching patterns accordingly ([Number dialects](../fleet/number-dialect.md)).
