# Trunks and outbound routes

## Trunks

Create / edit SIP trunks (ITSP or peer). Save → **Commit**.

Fleet design: **carriers usually live at the SBC / peering edge**, not as a fragile copy-per-node. Lab may still show instance trunks for solo trails.

## Outbound routes

Route: `/routes`

1. Open route → dial patterns / COS / trunk selection.
2. Save → **Commit**.

Test with a known extension after Commit, not only from the SPA.
