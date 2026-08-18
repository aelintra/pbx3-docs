# Install the Lab home PBX

**Audience:** a Windows-shop tech who can create Ubuntu VMs and paste a few commands. This is **VM B** in Lab deployment: pbx3 + pbx3api. No SIP, no public DNS, no Let’s Encrypt.

The [cloud/solo install](install-pbx3-pbx3api.md) page is the same stack with DNS and TLS. Use **this** page on a LAN VM.

Control host first: [Install the Lab control host](install-lab-control.md).

## What you need

| Have it? | Thing |
|:--------:|-------|
| ☐ | Ubuntu **24.04** VM with sudo and SSH |
| ☐ | This VM’s LAN IP (example lab: `192.168.1.31`) |
| ☐ | From a PC that can clone GitHub: `pbx3_0.0.5-5_all.deb`, the **pbx3api** tree, and **`pbx3/scripts/install-home-host.sh`** |
| ☐ | A site **Name** + admin email/password you invent (**8+** characters; not a docs placeholder like `admin@example.com`) |

**Skip CAGI** on ARM guests (and for this Lab install loop — no calls). Do **not** copy `pbx3cagi_*.deb`.

## On your PC — copy files to the VM

Put the installer, the `.deb`, and the API tree in one place on the VM (example: `/tmp/lab-home`):

```bash
scp pbx3/scripts/install-home-host.sh \
    pbx3/pbx3_0.0.5-5_all.deb \
    tech@192.168.1.31:/tmp/lab-home/
scp -r pbx3api tech@192.168.1.31:/tmp/lab-home/pbx3api
```

Use your VM user and LAN IP. Create `/tmp/lab-home` on the VM first if needed (`mkdir -p /tmp/lab-home`).

## On the home VM — one installer

```bash
cd /tmp/lab-home
chmod +x install-home-host.sh
sudo ./install-home-host.sh
```

Answer:

| Prompt | Example |
|--------|---------|
| Domain apex | `pbx3.com` (FQDN becomes `{shortuid}.pbx3.com` — no DNS required for Lab) |
| Site name | `Lab Home` |
| Admin SPA email | your address |
| Admin SPA password | 8+ characters |
| Fleet service token | from control `/etc/pbx3-gatekeeper/.env` (`PBX3_FLEET_SERVICE_TOKEN`) — Enter to skip |
| Org bucket | `lab-pbx3` (when token set) |
| SBC egress host | SBC LAN IP when doing SIP (Enter to skip until after Provision edge) |

The installer writes fleet API settings to `/opt/pbx3api/.env` when a token is provided (`PBX3_FLEET_MODE`, org bucket, optional egress host). If you set the SBC egress host, it also seeds the **Egress** trunk and restarts Asterisk (needs `pbx3-directory/tools/seed-fleet-egress-trunk.sh` on the VM or in the monorepo checkout).

Non-interactive example:

```bash
sudo PBX3_FLEET_SERVICE_TOKEN='…' PBX3_ORG_BUCKET=lab-pbx3 \
     PBX3_SBC_EGRESS_HOST=192.168.1.85 ./install-home-host.sh
```

## Prove it

On the VM (the installer also prints this):

```bash
curl -k -sS -o /dev/null -w "%{http_code}\n" https://127.0.0.1:44300/up
# expect 200
```

From your PC, run the SPA (`npm run dev`) and sign in with the admin email/password. API base for Vite proxy: `http://localhost:5173/api` after pointing `VITE_API_PROXY_TARGET` at this VM.

## Next

[Adopt this home into Fleet](install-lab-adopt.md) (SPA **Manage instance** → pick the row, or **Fleet console** → **Instances → Register instance**). Ops Mac onboard is **not** the Lab happy path.

OpenSIPS / calls are **not** part of this page.
