# Install the Lab SBC (SIP edge)

**Audience:** a MS or MacOS tech who can create Ubuntu VMs and paste a few commands.

## What we will install in this section.

This is **VM C**, the SBC Edge router — OpenSIPS + Filament admin on **amd64**. Control (`.33`) is already up. The home PBX comes **after** this page so the home installer can set Egress in one pass.

The PBX3 SBC edge router uses OpenSIPS.  OpenSIPS packages are **x86_64 only**. Do not use this section on ARM VMs.  If you have no option then you will need to install OpenSIPS from source and compile it.   You can find full instructions on the opensips website.

## What you need

| Have it? | Thing |
|:--------:|-------|
| ☐ | Ubuntu **24.04 amd64** VM with sudo and SSH (example lab: `192.168.1.85`) |
| ☐ | HTTPS to GitHub (public **pbx3sbc** + **pbx3sbc-admin**) |
| ☐ | A Filament admin email + password you invent (**10+** characters) — the installer will ask; do not put them in the paste |
| ☐ | Fleet service token from control `/etc/pbx3-gatekeeper/.env` (`PBX3_FLEET_SERVICE_TOKEN`) — the installer will ask. **Provision edge** is later, after you adopt the home. |

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

**Stop — do not paste passwords or tokens.** The block below is safe to copy as-is. The installer will **ask** for:

1. The OpenSIPS **database password** you invented in step 1 (typed twice)
2. A **Filament admin email** (a real address, not a docs hint)
3. A **Filament admin password** (10+ characters, typed twice)
4. The **fleet service token** from control `/etc/pbx3-gatekeeper/.env` (`PBX3_FLEET_SERVICE_TOKEN`) — paste that exact value. Do not invent a second token. **Provision edge** fails with `401 Unauthorized` if Filament’s token does not match control (and the home).

If `~/pbx3sbc-admin` already exists, skip `git clone` and only `cd ~/pbx3sbc-admin`.

```bash
cd ~
git clone --depth 1 https://github.com/aelintra/pbx3sbc-admin.git
cd ~/pbx3sbc-admin
sudo ./install.sh \
  --server-name 192.168.1.85 \
  --db-host localhost --db-name opensips --db-user opensips \
  --opensips-mi-url http://127.0.0.1:8888/mi \
  --admin-name Admin
```

`--server-name` is this VM’s LAN IP (no public DNS). Skip `--letsencrypt`.

Open **http://192.168.1.85/admin** (login with the Filament email/password you typed). Do not use snakeoil HTTPS.

If you skipped the fleet token at the prompt, set `PBX3_FLEET_SERVICE_TOKEN` in `~/pbx3sbc-admin/.env` before you **Provision edge** (after adopt).

## 3. Control already knows this IP

The [control installer](install-lab-control.md) prompt **SBC admin API URL** should already be `http://192.168.1.85/api`. If you pressed Enter earlier, re-run on the control VM (tokens are not rotated):

```bash
cd ~/pbx3/pbx3-directory
sudo PBX3_SBC_ADMIN_API_URL=http://192.168.1.85/api ./tools/install-control-host.sh
```

That's it for this VM for now. Do **not** Provision edge yet — there is no home in the catalog.

## Next

[Install the Lab home PBX](install-lab-home.md). Use this SBC LAN IP as the home **SBC egress host**.
