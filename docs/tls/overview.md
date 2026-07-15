# TLS and certificates overview

Exactly **one** active TLS identity for nginx and Asterisk, chosen in this order:

1. **Purchased (custom)** — `/opt/pbx3/etc/ssl/custom/fullchain.pem` + `privkey.pem` (**wins if present**)
2. **Let's Encrypt** — `/etc/letsencrypt/live/<primary>/`
3. **Snakeoil** — fallback for first boot / lab until LE

## Let's Encrypt model (Option A)

- One cert, **multi-SAN** = node FQDN + all tenant FQDNs
- **HTTP-01** only (no DNS API in product path)
- ~**50 SAN** LE practical limit
- Port **80** opened for challenge, then closed by scripts
- Auto renew via `/etc/cron.d/pbx3` (~03:17) using `le-renew-with-80.sh`

## Operator rule of thumb

| Change | Action |
|--------|--------|
| First cert on a node | Bootstrap / Get certificate |
| Tenant FQDN added or removed | **Sync with tenant list** |
| Calendar expiry only | Renew (cron or Renew now) |

Custom always overrides LE while the custom files exist — [Remove](purchased-certificate.md) to fall back.
