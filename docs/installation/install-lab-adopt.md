# Adopt a Lab home into Fleet

**Audience:** a MS or MacOS tech who can create Ubuntu VMs and paste a few commands. Control, SBC, and home PBX are already installed. This puts the home in the catalog, **Provision edge**, then you can add two extensions and make a call.

Sections **F** and **G** of the **[operator worksheet](install-lab-worksheet.md)** match this page.

## What we will do in this section.

Register the home in Fleet, Provision edge (dispatcher setid + Fail2ban whitelist), then create two extensions and point the phones at the **SBC** LAN IP.

Do **not** run Mac `onboard-fleet-instance.sh`.

## What you need

| Have it? | Thing |
|:--------:|-------|
| ☐ | [Control host](install-lab-control.md) health 200 |
| ☐ | [Home PBX](install-lab-home.md) `/up` 200 |
| ☐ | [Lab SBC](install-lab-sbc.md) Filament login works |
| ☐ | Fleet admin email/password from the control installer |
| ☐ | Home KSUID / FQDN from the home VM (below) |

On the **home** VM:

```bash
sudo sqlite3 /opt/pbx3/db/sqlite.db \
  "SELECT id, shortuid, fqdn, sitename FROM globals WHERE pkey='global';"
```

## In the SPA

1. Open the SPA from [Lab admin SPA (Vite)](install-lab-spa.md) → **Fleet console** (not the instance admin password).
2. Sign in with the **fleet** email/password from the control installer.
3. **Instances** — click **Register instance** (next to **Refresh**). If that button is missing, reload the tab once.
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

Leave Let’s Encrypt for later (cloud).

## Provision edge

Back in the SPA **Fleet console** (fleet email/password) → the Lab Home row → **Provision edge**. That assigns a dispatcher setid and whitelists the home IP on the SBC.

**Reusing the same SBC VM** across install tests: Provision edge also **removes stale fleet-owned domain rows** on the SBC that are no longer in the catalog (prior tenants or decommissioned instances). You do not need to reinit the SBC database or delete domains by hand. Optional check: **Fleet → Catalog reconcile** — orphan rows should be gone after Provision edge.

## Two phones and a call

1. Instance admin → a tenant → two extensions (example **101** / **102**). **Save**, then **Commit** (extensions are not in Asterisk until Commit).
2. Point both phones at the **SBC** LAN IP (example `192.168.1.85`), not the home `.31`.

**Order matters on fleet:** **Commit before** aiming phones at the SBC. If a phone registers earlier (e.g. reused lab handset still on **`.85`** from a prior tenant), Asterisk may log `AOR '' not found for endpoint 'Egress'` until Commit and correct auth — benign; see **`FLEET_COMMIT_RELOAD_REQUIREMENTS.md`** **#5j-a**.

3. Place a call 101 ↔ 102. CAGI must already be on the home ([ARM compile](install-lab-home.md#cagi-needed-for-a-call)).

That's the Lab walk.
