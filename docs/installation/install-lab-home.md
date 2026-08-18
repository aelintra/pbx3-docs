# Install the Lab home PBX

**Audience:** a Windows-shop tech who can create Ubuntu VMs and paste a few commands. This is **VM B** in Lab deployment: pbx3 + pbx3api. No SIP, no public DNS, no Let’s Encrypt.

The [cloud/solo install](install-pbx3-pbx3api.md) page is the same stack with DNS and TLS. Use **this** page on a LAN VM.

Control host first: [Install the Lab control host](install-lab-control.md).

## What you need

| Have it? | Thing |
|:--------:|-------|
| ☐ | Ubuntu **24.04** VM with sudo and SSH |
| ☐ | This VM’s LAN IP (example lab: `192.168.1.31`) |
| ☐ | HTTPS to GitHub (**pbx3api** is public; **pbx3** copy the tree if clone asks for login) |
| ☐ | A site **Name** + admin email/password you invent (**8+** characters; not a docs placeholder like `admin@example.com`) |

**Installer loop:** skip CAGI (`PBX3_INSTALL_CAGI` unset). **pbx3cagi** is still private — do not expect an on-box clone during home install.

**When you want AGI on an ARM home:** compile on the guest (do not copy a Mac binary). Rsync the `pbx3cagi` tree, then:

```bash
sudo apt-get install -y make libbsd-dev
cd ~/pbx3cagi/pbx3cagi-1.0.0/csource
make clean && make
sudo cp -a pbx3cagi /usr/share/asterisk/agi-bin/pbx3cagi.arm64
sudo chmod 755 /usr/share/asterisk/agi-bin/pbx3cagi.arm64
sudo ln -sfn pbx3cagi.arm64 /usr/share/asterisk/agi-bin/pbx3cagi
file /usr/share/asterisk/agi-bin/pbx3cagi.arm64   # ELF aarch64
```

On amd64, the installer path is: clone **pbx3cagi** and set `PBX3_INSTALL_CAGI=1` (or `apt install` the `_all.deb`).

## On the home VM

Paste this (public **pbx3** + **pbx3api**; run the installer from the pbx3 tree so the `.deb` on `main` is found):

```bash
sudo apt-get update
sudo apt-get install -y git
cd ~
git clone --depth 1 https://github.com/aelintra/pbx3.git
git clone --depth 1 https://github.com/aelintra/pbx3api.git pbx3/pbx3api
cd pbx3
sudo ./scripts/install-home-host.sh
```

**pbx3api** is public. If the **pbx3** clone asks for a GitHub login, copy `pbx3/` (including `scripts/install-home-host.sh` and the `pbx3_*.deb` on `main`) from a machine that already has it, clone **pbx3api** into `pbx3/pbx3api`, then run the installer.

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

## Next

1. [Install the Lab admin SPA (Vite)](install-lab-spa.md) on your PC (`npm run dev` → http://localhost:5173).
2. [Adopt this home into Fleet](install-lab-adopt.md). Ops Mac onboard is **not** the Lab happy path.

OpenSIPS / calls are **not** part of this page.
