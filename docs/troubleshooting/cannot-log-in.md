# Cannot log in

| Symptom | Check |
|---------|-------|
| Empty catalog / load fail | Public catalog URL; key `catalog/instance-index.json`; CORS for SPA origin; BPA |
| API “Load failed” | SG **44300**; Shorewall; `curl -k https://NODE:44300/up` → 200 |
| Browser TLS warning | Node still snakeoil / wrong SAN — [First LE](../tls/first-letsencrypt.md) |
| Solo confusion | Clear catalog URL; use API URL only |
| Fleet gate stuck | Gatekeeper up (`/health`); fleet user enabled; Exit then Sign in again |
| Break-glass only works | Prefer email/password; rotate/revoke pasted tokens after ops |

Lab gatekeeper: `https://control.pbx3.com` · golden API: `https://08jzwn.pbx3.com:44300/api`.
