# Install the Lab control host

**Audience:** a Windows-shop tech who can create Ubuntu VMs and paste a few commands. One installer on this box, then a browser.

This is **VM A** in Lab deployment: Gatekeeper + Garage (org catalog). It does **not** install the home PBX (that is the other VM) and it does **not** require SIP.

Solo PBX (one box, no fleet) stays on [Install pbx3 and pbx3api](install-pbx3-pbx3api.md) — no control host, no Garage.

## What you need

| Have it? | Thing |
|:--------:|-------|
| ☐ | Ubuntu **24.04** VM with sudo and SSH |
| ☐ | This VM’s LAN IP (example lab: `192.168.1.33`) |
| ☐ | HTTPS to GitHub (public clones; no GitHub login). Control installer is `pbx3-directory/` in **pbx3** |
| ☐ | A fleet admin email + password you invent (**10+** characters) |

You will **not** need AWS, Garage keys, or a Mac.

## On the control VM

Paste this (creates `~/pbx3/pbx3-directory` and runs the installer):

```bash
sudo apt-get update
sudo apt-get install -y git
cd ~
git clone --depth 1 https://github.com/aelintra/pbx3.git
cd pbx3/pbx3-directory
sudo ./tools/install-control-host.sh
```

Answer:

| Prompt | Example |
|--------|---------|
| Fleet slug | `lab` (bucket becomes `lab-pbx3`) |
| This VM LAN IP | `192.168.1.33` (installer usually detects it) |
| Fleet admin email | your address |
| Fleet admin password | 10+ characters |
| SBC admin API URL | `http://192.168.1.85/api` when SIP lab has an SBC — Enter if not yet |

When it finishes, open in a browser:

- Health: `http://192.168.1.33/health` → `{"status":"ok"}`
- Catalog: `http://192.168.1.33/catalog/instance-index.json` → `"instances": []` until you adopt a home

The installer also enables **`pbx3-fleet-probe.timer`** (home `/up` every minute so Fleet stays **Healthy** after adopt). Keys stay on the box. The finish banner prints **`PBX3_FLEET_SERVICE_TOKEN`** and org bucket for the home / SBC installers.

SPA **Fleet** mode talks to `http://192.168.1.33` with the email/password you set.

Re-run is safe with the same answers (it will not rotate tokens).

## Next

1. Home VM: [Install the Lab home PBX](install-lab-home.md).
2. Your PC: [Install the Lab admin SPA (Vite)](install-lab-spa.md).
3. [Adopt the home](install-lab-adopt.md) from Fleet (**Instances → Register instance**).

OpenSIPS / calls are **not** part of this page.
