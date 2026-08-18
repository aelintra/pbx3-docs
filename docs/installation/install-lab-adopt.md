# Adopt a Lab home into Fleet

**Audience:** same Windows-shop tech. Control host and home PBX are already installed. This puts the home in the catalog so the SPA picker has a row.

Do **not** run Mac `onboard-fleet-instance.sh`. Do **not** Provision edge (no SIP in this Lab loop).

## What you need

| Have it? | Thing |
|:--------:|-------|
| ☐ | [Control host](install-lab-control.md) health 200 |
| ☐ | [Home PBX](install-lab-home.md) `/up` 200 |
| ☐ | Fleet admin email/password from the control installer |
| ☐ | Home KSUID / FQDN from the home VM (below) |

On the **home** VM:

```bash
sqlite3 /opt/pbx3/db/sqlite.db \
  "SELECT id, shortuid, fqdn, sitename FROM globals WHERE pkey='global';"
```

## In the SPA

1. Open the SPA from [Lab admin SPA (Vite)](install-lab-spa.md) → **Fleet console** (not the instance admin password).
2. Sign in with the **fleet** email/password from the control installer.
3. **Instances → Register instance**.
4. Fill:

| Field | Lab example |
|-------|-------------|
| Instance id (KSUID) | from sqlite `id` |
| FQDN | from sqlite `fqdn` (e.g. `n8hgxq.pbx3.com`) |
| API base URL | `https://192.168.1.31:44300/api` (this VM’s LAN IP) |
| Label | same as sitename (e.g. `Lab Home`) |
| Environment | `lab` |
| SBC dispatcher setid | leave unset |
| Skip /up verify | only if Register fails on snakeoil |

5. Submit. Catalog `http://192.168.1.33/catalog/instance-index.json` should list the row.

## Then pick it

Log out. On the instance login screen, **Manage instance** → pick **Lab Home** (use **Refresh catalog** on that picker if the row is missing) → sign in with the **home** admin email/password (not the fleet user).

Leave Let’s Encrypt and Provision edge for later (cloud / SIP).
