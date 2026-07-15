# Certificate / HTTPS errors

| Symptom | Fix |
|---------|-----|
| Still snakeoil | DNS missing; port **80** blocked (SG **and** Shorewall); run bootstrap / Get certificate |
| Hostname mismatch after new tenant | DNS → node; Certificates → **Sync** (not Renew) |
| After restore / rebuild | Sync + DNS for all SANs |
| Custom “stuck” over LE | Remove purchased cert |
| HTTP-01 fail | Open 80 temporarily; `curl` ACME path; retry bootstrap |

Trusted check:

```bash
curl -sS -o /dev/null -w '%{http_code}\n' https://08jzwn.pbx3.com:44300/up
```
