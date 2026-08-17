# Install pbx3 and pbx3api

Bring up a **clean Ubuntu 24.04** node from first principles: get packages → install → identity → API → DNS → TLS → prove `/up`.

This page stops at a working **solo** instance (Admin SPA login). To join an existing fleet (IAM, catalog, Egress, Provision edge), continue with [Commission a fleet instance](../fleet/commission-instance.md) from onboard onward, or [Onboard](../fleet/onboard-instance.md) if the node is already installed.

!!! tip "Fleet greenfield end-to-end"
    EC2 launch through SBC edge is one path: [Commission a fleet instance](../fleet/commission-instance.md). This page is the **stack install** story (solo or the install half of commission).

## What you are installing

| Piece | Form | Lands on the node as |
|-------|------|----------------------|
| **pbx3** | `.deb` | `/opt/pbx3` — Asterisk configs, SQLite, scripts |
| **pbx3cagi** | `.deb` | CAGI (call logic) — install with pbx3 |
| **pbx3api** | **git tree** (no `.deb`) | `/opt/pbx3api` — nginx + PHP-FPM API on **:44300** |

There is **no** monorepo. GitHub has separate repos. Release `.deb`s live at the **root of `pbx3` and `pbx3cagi` on `main`** today (private repos — clone from a laptop that already has GitHub auth).

---

## Before you start

| Have it? | Thing |
|:--------:|-------|
| ☐ | Ubuntu **24.04** node with sudo (bare metal, VM, or EC2) |
| ☐ | SSH access (`ubuntu@…` or equivalent) + private key if needed |
| ☐ | Laptop with **git**, **scp**, and GitHub access to **`aelintra/pbx3`**, **`aelintra/pbx3cagi`**, **`aelintra/pbx3api`** |
| ☐ | Inbound **22**, **80** (LE), **44300**; outbound **443** — see [Requirements](requirements.md) |
| ☐ | DNS control for an apex (e.g. `pbx3.com`) **or** readiness to create an **A** record after install |
| ☐ | Email for Let’s Encrypt + friendly site **Name** + admin SPA email/password you invent |

**Lab packages (adjust when newer):** `pbx3_0.0.5-5_all.deb` · `pbx3cagi_1.0.0-18_all.deb`.

### Worksheet

Use **straight** quotes only. Put comments on their **own** lines — zsh often treats `# …` on the same line as `export` as more arguments (so `# or user@host` becomes `export: not valid in this context: user@host`).

```bash
# SSH key path (omit KEY_FILE later if your agent already has the key)
export KEY_FILE=/path/to/your.pem

# SSH target
export SSH_HOST=ubuntu@YOUR.PUBLIC.IP

export LE_EMAIL=you@example.com

# Friendly Name — not the hostname
export SITE_NAME='My node'

# Apex; installer mints opaque shortuid → fqdn={shortuid}.{apex}
export DOMAIN_TLD=pbx3.com

# After installer (Step 4), from sqlite fqdn column, e.g. 7k2m9q.pbx3.com:
# export INSTANCE_FQDN=…
```

---

## Step 1 — Clone package repos on the laptop

On your **ops laptop** (not the PBX node):

```bash
mkdir -p ~/GiT/pbx3-master
cd ~/GiT/pbx3-master

git clone https://github.com/aelintra/pbx3.git
git clone https://github.com/aelintra/pbx3cagi.git
# pbx3api is cloned onto the node in Step 5 (or scp’d if the node has no GitHub access)

cd pbx3 && git checkout main && git pull --ff-only
cd ../pbx3cagi && git checkout main && git pull --ff-only

export PBX3_DEB=~/GiT/pbx3-master/pbx3/pbx3_0.0.5-5_all.deb
export CAGI_DEB=~/GiT/pbx3-master/pbx3cagi/pbx3cagi_1.0.0-18_all.deb
test -f "$PBX3_DEB" && test -f "$CAGI_DEB" && echo "debs ok"
```

The parent folder name (`pbx3-master`) is arbitrary. Point the `export`s at wherever you cloned.

!!! note "Why not clone debs on the node?"
    While **pbx3** / **pbx3cagi** are private, every new box would need deploy keys. Copying release `.deb`s from a laptop that already authenticates to GitHub is the usual lab path. When those repos are public, on-node `git clone` + build becomes fine.

---

## Step 2 — Copy `.deb`s to the node

```bash
scp -i "$KEY_FILE" "$PBX3_DEB" "$CAGI_DEB" "${SSH_HOST}:/tmp/"
# omit -i "$KEY_FILE" if your SSH agent / default key already works
```

---

## Step 3 — Node: OS prep and install packages

SSH in, then:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y ssmtp sqlite3 curl ca-certificates git
sudo chmod +x /etc/ssmtp

cd /tmp
sudo apt install -y ./pbx3_0.0.5-5_all.deb ./pbx3cagi_1.0.0-18_all.deb
dpkg-query -W pbx3 pbx3cagi
```

`apt install` places files under `/opt/pbx3`. It does **not** create the customer DB — that is the next step.

---

## Step 4 — First-run installer (identity)

You do **not** type the FQDN. You supply the **apex**; the installer mints an opaque **shortuid** and sets `fqdn = {shortuid}.{apex}`.

| You provide | Installer writes |
|-------------|------------------|
| `DOMAIN_TLD` (e.g. `pbx3.com`) | `shortuid`, `fqdn` = `{shortuid}.pbx3.com` |
| `INSTANCE_SITENAME` | Friendly **Name** on Home |
| Admin email + password | First SPA user (no factory password) |
| — | `globals.id` (**KSUID**) |

```bash
sudo DOMAIN_TLD="${DOMAIN_TLD}" \
  INSTANCE_SITENAME="${SITE_NAME}" \
  PBX3_ADMIN_EMAIL=ops@example.com \
  PBX3_ADMIN_PASSWORD='choose-a-strong-password' \
  /opt/pbx3/scripts/installer.sh
```

Omit `PBX3_ADMIN_*` to answer prompts on a real TTY. Details: [First-run installer](first-run.md).

**Do not** pass vanity `INSTANCE_FQDN=kildare.pbx3.com` unless you intentionally set `PBX3_ALLOW_VANITY_FQDN=1`.

Verify an admin was created (installer end banner also reports this on current `installer.sh`):

```bash
sqlite3 /opt/pbx3/db/sqlite.db "SELECT id,email FROM users;"
# must show at least one row
```

If empty (older packages called bootstrap with `/bin/sh` and silently failed), create one:

```bash
sudo PBX3_ADMIN_EMAIL='you@yourdomain.com' \
  PBX3_ADMIN_PASSWORD='choose-a-strong-password' \
  /opt/pbx3/scripts/bootstrap-admin-user.sh
# or interactive on a real TTY:
# sudo /opt/pbx3/scripts/bootstrap-admin-user.sh
```

### Required — record identity before DNS / LE

The installer mints these; they are **not** the SSH nickname (e.g. `virginia1.pbx3.com`). Newer installer builds print a final **Instance identity** block; always capture them either way:

```bash
# On the node — shows KSUID | shortuid | fqdn | sitename
sqlite3 /opt/pbx3/db/sqlite.db \
  "SELECT id, shortuid, fqdn, sitename FROM globals WHERE pkey='global';"

# Export into this shell (and your Mac worksheet) — required before dig / LE
export KSUID=$(sqlite3 /opt/pbx3/db/sqlite.db "SELECT id FROM globals WHERE pkey='global';")
export SHORTUID=$(sqlite3 /opt/pbx3/db/sqlite.db "SELECT shortuid FROM globals WHERE pkey='global';")
export INSTANCE_FQDN=$(sqlite3 /opt/pbx3/db/sqlite.db "SELECT fqdn FROM globals WHERE pkey='global';")
echo "KSUID=$KSUID"
echo "SHORTUID=$SHORTUID"
echo "INSTANCE_FQDN=$INSTANCE_FQDN"
```

Do **not** continue to Step 7 until `echo "$INSTANCE_FQDN"` prints a real name (e.g. `xhcjkh.pbx3.com`). Empty → `dig` fails with `'' is not a legal name`.

!!! warning
    Do **not** run `reloader.sh` after a clean first install.  
    Do **not** set fleet `PBX3_ORG_BUCKET` yet if you plan to [onboard](../fleet/onboard-instance.md) later — the onboard script owns that.

---

## Step 5 — Deploy pbx3api

pbx3api is a **git checkout** at `/opt/pbx3api`, then its installer.

### Prefer: clone on the node

```bash
sudo rm -rf /opt/pbx3api
sudo git clone https://github.com/aelintra/pbx3api.git /opt/pbx3api
cd /opt/pbx3api
sudo git checkout main
sudo git pull --ff-only origin main
```

If the repo is private and the node has no credentials, clone on the laptop and copy:

```bash
# laptop
git clone https://github.com/aelintra/pbx3api.git /tmp/pbx3api
scp -i "$KEY_FILE" -r /tmp/pbx3api "${SSH_HOST}:/tmp/pbx3api"
# node
sudo rm -rf /opt/pbx3api
sudo mv /tmp/pbx3api /opt/pbx3api
```

### Run the API installer

Drop nginx’s default site if it will fight for port **80** (needed for Let’s Encrypt):

```bash
sudo rm -f /etc/nginx/sites-enabled/default
sudo /opt/pbx3api/scripts/installer.sh
```

API listens on **`https://<host>:44300`** (snakeoil until Step 7).

Optional tidy:

```bash
cd /opt/pbx3api
sudo composer install --no-dev
sudo php artisan config:clear
```

---

## Step 6 — Prove the stack (before DNS)

**On the node**, use loopback — not the public FQDN:

```bash
curl -k -sS -o /dev/null -w "%{http_code}\n" https://127.0.0.1:44300/up
# expect 200

sudo systemctl is-active nginx php8.3-fpm asterisk
```

**Stop if not 200.** DNS and LE will not fix a broken API.

!!! warning "AWS hairpin — do not curl the public FQDN from the node"
    On EC2, `curl https://$INSTANCE_FQDN:44300/up` **from the instance itself** usually **times out**. The box cannot open TCP to its own public IP/EIP. That is normal.

    | Where | Command |
    |-------|---------|
    | **On the node** | `https://127.0.0.1:44300/up` (use `-k` until LE) |
    | **Ops laptop / Mac** | `https://${INSTANCE_FQDN}:44300/up` |

    Also open **AWS security group TCP 44300** to your **ops public IP `/32`** (and Gatekeeper if fleet). Port **80** `0.0.0.0/0` only covers Let’s Encrypt — it does **not** open the API.

If nginx complains about a duplicate `default_server` on port 80:

```bash
sudo rm -f /etc/nginx/sites-enabled/default
sudo nginx -t && sudo systemctl reload nginx
```

---

## Step 7 — DNS

Requires **`INSTANCE_FQDN`** from [Step 4](#required--record-identity-before-dns--le). If unset, stop and export it — do not dig an empty name.

| Name | Type | Value |
|------|------|--------|
| Exact `globals.fqdn` (e.g. `xhcjkh.pbx3.com`) | **A** | Node public IP / EIP |

```bash
test -n "$INSTANCE_FQDN" || { echo "INSTANCE_FQDN not set — re-run Step 4 exports"; exit 1; }
echo "Checking DNS for $INSTANCE_FQDN"
dig +short "$INSTANCE_FQDN"
# must equal this node’s public IP
```

Fleet nodes do **not** publish public tenant A records. Solo multi-tenant Option A is a different TLS story — see [TLS overview](../tls/overview.md).

---

## Step 8 — First Let’s Encrypt certificate

Port **80** must reach this host from the internet. DNS A must already match.

```bash
sudo /opt/pbx3/scripts/le-instance-bootstrap.sh "$LE_EMAIL"
# staging: sudo PBX3_LE_STAGING=1 /opt/pbx3/scripts/le-instance-bootstrap.sh "$LE_EMAIL"
```

Or SPA → **Certificates → Get certificate** after snakeoil login. Full notes: [First Let's Encrypt certificate](../tls/first-letsencrypt.md).

**Prove trusted HTTPS from the ops laptop** (not from the node — see hairpin warning in Step 6). SG must allow your Mac’s public IP on **44300**:

```bash
# On the Mac / ops workstation
curl -4 -sS --connect-timeout 5 --max-time 10 -o /dev/null -w "%{http_code}\n" "https://${INSTANCE_FQDN}:44300/up"
# expect 200 without -k

# On the node only:
curl -k -sS -o /dev/null -w "%{http_code}\n" https://127.0.0.1:44300/up
```

Open the Admin SPA **from the laptop** and [sign in](../getting-started/sign-in.md) with the email/password from Step 4. API base URL example: `https://${INSTANCE_FQDN}:44300/api`.

---

## Checklist

- [ ] Cloned `pbx3` + `pbx3cagi` on laptop; `.deb`s present
- [ ] Copied debs to node; `apt install` succeeded
- [ ] `installer.sh` created KSUID / opaque FQDN / admin user
- [ ] pbx3api at `/opt/pbx3api`; API installer run
- [ ] `GET https://127.0.0.1:44300/up` → **200** (on node)
- [ ] DNS **A** for `globals.fqdn` → public IP
- [ ] Let’s Encrypt applied
- [ ] From **ops laptop**: trusted `https://{fqdn}:44300/up` → **200** (SG allows your IP on 44300)
- [ ] Admin SPA login works from the laptop

## Failure cheat sheet

| Symptom | Likely fix |
|---------|------------|
| On node, `curl https://$INSTANCE_FQDN:44300/up` hangs | Expected hairpin — use `127.0.0.1` on the node; public URL from the laptop |
| Laptop curl to `:44300` times out (~5–130s) | SG: add your public IP `/32` on **44300** (port 80 open is not enough) |
| LE fails `example.com` contact | Use a real email — Let’s Encrypt rejects `*.example.com` |
| `/up` not 200 on localhost | pbx3api installer, nginx default site, PHP-FPM |
| SPA login fails / `users` empty | Run `bootstrap-admin-user.sh` (see Step 4). Fixed in installer: must call bootstrap with **bash**, not `sh` |
## Next

| Goal | Go to |
|------|--------|
| Use the box alone | [Solo trial](../getting-started/solo-trial.md) · [Admin guide](../admin/home-dashboard.md) |
| Join an **existing** fleet | [Onboard a second instance](../fleet/onboard-instance.md) |
| New fleet home including EC2 + edge | [Commission a fleet instance](../fleet/commission-instance.md) |
| Same KSUID after EC2 loss | [Rebuild from S3](../fleet/rebuild-from-s3.md) |

## Related

- [Requirements](requirements.md) · [First-run installer](first-run.md) · [Upgrade a package](upgrade.md)
