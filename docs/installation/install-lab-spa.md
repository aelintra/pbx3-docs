# Install the Lab admin SPA (Vite)

**Audience:** a MS or MacOS tech who can create Ubuntu VMs and paste a few commands.  This runs on **your PC**, not on a VM. The home API is already up ([Lab home PBX](install-lab-home.md) `/up` → 200).

## What we will install in this section.

PBX3 Fleet management and admin uses a Single Page App (SPA).  It runs under npm (Node.js) on **your PC**, not on a VM. The Fleet API is already up if you have completed the previous sections in this install sequence.

Until a published GitHub Pages bundle exists, Lab uses the **Vite dev server** so the browser URL stays on `http://localhost:5173`. Usually, you will run npm in your VSC or Cursor IDE. You can find specific Node.js install instructions for your setup online.


## What you need

| Have it? | Thing |
|:--------:|-------|
| ☐ | **Node.js LTS** (includes **npm**) — [nodejs.org](https://nodejs.org/) if `node -v` fails. Vite wants Node **18+** (20 LTS is fine). |
| ☐ | Git + HTTPS to GitHub (public **pbx3spa**) |
| ☐ | Home LAN IP (PBX example `192.168.1.31`) and control LAN IP (example `192.168.1.33`) |

## Clone and env

```bash
git clone --depth 1 https://github.com/aelintra/pbx3spa.git
cd pbx3spa
```

Create **`.env.development`** in that folder (this file is not in git). Edit the two IPs if yours differ:  In our example .31 is the PBX instance you created and .33 is the Gatekeeper/Garage instance.

```env
VITE_API_PROXY_TARGET=https://192.168.1.31:44300
VITE_DEFAULT_API_BASE_URL=http://localhost:5173/api

VITE_CATALOG_PROXY_TARGET=http://192.168.1.33
VITE_INSTANCE_DIRECTORY_URL=/dev-catalog/catalog/instance-index.json

VITE_FLEET_GATEKEEPER_PROXY_TARGET=http://192.168.1.33
VITE_FLEET_GATEKEEPER_URL=/fleet-gk
```

That tells Vite:

| Variable | What it does |
|----------|----------------|
| `VITE_API_PROXY_TARGET` | Browser `/api` → home `https://…:44300` (TLS not checked by the proxy — Lab snakeoil is OK) |
| `VITE_DEFAULT_API_BASE_URL` | Login form pre-fill: `http://localhost:5173/api` |
| `VITE_CATALOG_PROXY_TARGET` + `VITE_INSTANCE_DIRECTORY_URL` | Fleet instance picker via Garage catalog on the control host |
| `VITE_FLEET_GATEKEEPER_*` | **Fleet console** login → Gatekeeper on the control host |

## Run Vite

```bash
npm install
npm run dev
```

Open **http://localhost:5173** in the browser. Leave that terminal running. Stop with Ctrl+C.

Do **not** open `https://192.168.1.31:44300` in the browser (snakeoil). Always use the Vite URL.

## Sign in

**Instance admin** (tenants, extensions, Commit): email/password from the [home installer](install-lab-home.md). Leave API base as `http://localhost:5173/api`.

**Fleet console** (catalog, Register instance): [control installer](install-lab-control.md) fleet email/password. Then [adopt the home](install-lab-adopt.md).

Modes use different passwords. Do not mix them.

That's it for this section.  You're almost there...

## Next

[Adopt a Lab home into Fleet](install-lab-adopt.md).
