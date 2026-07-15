# Solo trial (one node)

Run **one** PBX3 instance without fleet catalog, org S3, Gatekeeper, or SBC.

## What you need

| Need | Do not need |
|------|-------------|
| Ubuntu 24.04 node with `pbx3` + `pbx3api` | Org S3 bucket / instance catalog |
| Admin SPA (local or Pages) | Multi-instance picker |
| Reachable API at `https://{fqdn}:44300/api` | Control host / Gatekeeper |
| Sanctum admin login | SBC (optional for internet SIP later) |

## Sign in

1. Open the admin SPA.
2. Enter **email** and **password** (instance admin).
3. Enter **API base URL**, e.g. `https://YOURNODE.pbx3.com:44300/api`.
4. Do **not** configure a catalog URL (fleet picker stays unused).

## Lab tip

Golden example API:

`https://08jzwn.pbx3.com:44300/api`

Verify TLS without the SPA proxy:

```bash
curl -sS -o /dev/null -w '%{http_code}\n' https://08jzwn.pbx3.com:44300/up
# expect 200 (no -k)
```

When you outgrow solo, see [Fleet overview](../fleet/overview.md) and [Onboard a second instance](../fleet/onboard-instance.md).
