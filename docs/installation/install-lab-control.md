# Install the Lab control host

**Audience:** a Windows-shop tech who can create Ubuntu VMs and paste a few commands. One installer on this box, then a browser.

This is **VM A** in Lab deployment: Gatekeeper + Garage (org catalog). It does **not** install the home PBX (that is the other VM) and it does **not** require SIP.

Solo PBX (one box, no fleet) stays on [Install pbx3 and pbx3api](install-pbx3-pbx3api.md) — no control host, no Garage.

## What you need

| Have it? | Thing |
|:--------:|-------|
| ☐ | Ubuntu **24.04** VM with sudo and SSH |
| ☐ | This VM’s LAN IP (example lab: `192.168.1.33`) |
| ☐ | The **pbx3-directory** tree on the VM (git clone, or one copy from your PC) |
| ☐ | A fleet admin email + password you invent (**10+** characters) |

You will **not** need AWS, Garage keys, or a Mac.

## On the control VM

```bash
cd pbx3-directory
sudo ./tools/install-control-host.sh
```

Answer:

| Prompt | Example |
|--------|---------|
| Fleet slug | `lab` (bucket becomes `lab-pbx3`) |
| This VM LAN IP | `192.168.1.33` (installer usually detects it) |
| Fleet admin email | your address |
| Fleet admin password | 10+ characters |

When it finishes, open in a browser:

- Health: `http://192.168.1.33/health` → `{"status":"ok"}`
- Catalog: `http://192.168.1.33/catalog/instance-index.json` → `"instances": []` until you adopt a home

SPA **Fleet** mode talks to `http://192.168.1.33` with the email/password you set. Keys stay on the box.

Re-run is safe with the same answers (it will not rotate tokens).

## Next

1. Home VM: [Install pbx3 and pbx3api](install-pbx3-pbx3api.md) (skip CAGI on ARM guests).
2. Adopt the home from Fleet (panel when shipped; ops CLI is not the Lab happy path).

OpenSIPS / calls are **not** part of this page.
