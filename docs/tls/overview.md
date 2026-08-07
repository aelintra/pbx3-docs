# TLS and certificates overview

!!! danger "DO NOT create DNS A records for tenant domains"
    On an **SBC fleet**, names like `{shortuid}.pbx3.com` are **SIP domains only** — not hostnames to publish in DNS.

    **Do not** create public **A** / AAAA records for tenants. Phones use the **SBC**; the SPA uses the **instance** URL from the catalog. Publish **A** records only for **instance**, **SBC**, and **control** hostnames.

    Creating tenant A records causes wrong Let's Encrypt Sync behaviour and move confusion.

Exactly **one** active TLS identity for nginx and Asterisk, chosen in this order:

1. **Purchased (custom)** — `/opt/pbx3/etc/ssl/custom/fullchain.pem` + `privkey.pem` (**wins if present**)
2. **Let's Encrypt** — `/etc/letsencrypt/live/<primary>/`
3. **Snakeoil** — fallback for first boot / lab until LE

## SBC fleet (product default)

- Node LE = **instance FQDN only** (HTTP-01)
- **No** public tenant A records (SIP domain string + SBC `domain`/setid)
- SPA admin uses **instance** URL
- WSS host = edge/instance — not tenant FQDN

## Solo / direct-to-node (Option A)

- One cert, **multi-SAN** = node FQDN + tenant FQDNs that resolve here
- **HTTP-01** only (no DNS API in product path)
- ~**50 SAN** LE practical limit

Port **80** opened for challenge, then closed by scripts. Auto renew via `/etc/cron.d/pbx3` (~03:17) using `le-renew-with-80.sh`.

## Operator rule of thumb

| Change | Action |
|--------|--------|
| First cert on a node | Bootstrap / Get certificate |
| Fleet: refresh instance cert | **Sync certificate** |
| Solo: tenant FQDN added/removed | **Sync certificate** (with tenant As pointing here) |
| Calendar expiry only | Renew (cron or Renew now) |

Custom always overrides LE while the custom files exist — [Remove](purchased-certificate.md) to fall back.
