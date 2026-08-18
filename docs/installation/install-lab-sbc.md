# Install the Lab SBC (SIP edge)

**Audience:** same Windows-shop tech. This is **VM C** — OpenSIPS + Filament admin on **amd64**. Control (`.33`) and home (`.31`) are already up ([adopt](install-lab-adopt.md) first).

OpenSIPS packages are **x86_64 only**. Do not install this on the ARM control or home VMs.

## What you need

| Have it? | Thing |
|:--------:|-------|
| ☐ | Ubuntu **24.04 amd64** VM with sudo and SSH (example lab: `192.168.1.85`) |
| ☐ | HTTPS to GitHub (public **pbx3sbc** + **pbx3sbc-admin**) |
| ☐ | A Filament admin email + password you invent (**10+** characters) |
| ☐ | Fleet service token from control `/etc/pbx3-gatekeeper/.env` (`PBX3_FLEET_SERVICE_TOKEN`) — needed for **Provision edge** |

No Let’s Encrypt on this Lab path (HTTP on the LAN IP).

## 1. Edge (OpenSIPS)

On the SBC VM:

```bash
sudo apt-get update
sudo apt-get install -y git
cd ~
git clone --depth 1 https://github.com/aelintra/pbx3sbc.git
cd pbx3sbc
sudo ./install.sh --advertised-ip 192.168.1.85 --preferlan
```

When asked **Reinitialize database? (y/N):** answer **`y`** on a fresh VM. Invent a DB password and keep it for the admin installer.

Check: `systemctl is-active opensips mariadb` → both `active`.

## 2. Admin (Filament)

```bash
cd ~
git clone --depth 1 https://github.com/aelintra/pbx3sbc-admin.git
cd pbx3sbc-admin
sudo ./install.sh \
  --server-name 192.168.1.85 \
  --db-host localhost --db-name opensips --db-user opensips \
  --db-password '<same DB password>' \
  --opensips-mi-url http://127.0.0.1:8888/mi \
  --admin-name Admin \
  --admin-email '<your address>' \
  --admin-password '<10+ characters>' \
  --fleet-service-token '<from control .env>'
```

`--server-name` is the LAN IP (no public DNS). Skip `--letsencrypt`.

Open **http://192.168.1.85/admin** (login with the Filament email/password). Do not use snakeoil HTTPS.

If you skipped the fleet token, set `PBX3_FLEET_SERVICE_TOKEN` in `~/pbx3sbc-admin/.env` before Provision edge.

## 3. Control already knows this IP

The [control installer](install-lab-control.md) prompt **SBC admin API URL** should be `http://192.168.1.85/api`. If you pressed Enter earlier, re-run on the control VM (tokens are not rotated):

```bash
cd ~/pbx3/pbx3-directory
sudo PBX3_SBC_ADMIN_API_URL=http://192.168.1.85/api ./tools/install-control-host.sh
```

## 4. Provision edge + Egress

In the SPA **Fleet console** → the Lab Home row → **Provision edge** (setid assigned; home IP is Fail2ban-whitelisted).

Then on the **home** VM, seed the Egress trunk if you skipped it at home install:

```bash
sudo PBX3_SBC_EGRESS_HOST=192.168.1.85 \
  ~/pbx3/pbx3-directory/tools/seed-fleet-egress-trunk.sh /opt/pbx3/db/sqlite.db
sudo /opt/pbx3/scripts/genAst.sh
```

If `pbx3-directory` is not on the home VM, copy that tools file from the control tree (or the laptop checkout) and run the same two commands. Do not re-run `install-home-host.sh` just to seed Egress.

Desk phones are **not** this page. ARM home CAGI is compile-on-guest — see [Install the Lab home PBX](install-lab-home.md).
