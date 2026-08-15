# Commission a fleet instance

Bring up a **new home box** (EC2 / node) and join it to the fleet: install → DNS/LE → onboard → SBC edge → smoke.

This creates a **new KSUID**. It is **not** the same as [Rebuild a fleet node from S3](rebuild-from-s3.md) (same identity after EC2 loss).

| Object | What you create |
|--------|-----------------|
| **Instance** (Asterisk node) | New EC2 + packages + catalog row |
| **Edge membership** | Dispatcher setid + Asterisk Peer via **Provision edge** |
| **Tenant** (optional later) | Fleet → **Tenants** → Create (needs setid first) |

!!! tip "Already installed, only need fleet join?"
    Jump to [Onboard a second instance](onboard-instance.md) (Act 2 only).  
    Retiring a box → [Decommission a fleet instance](decommission-instance.md).

## Before you start

| Have it? | Thing |
|:--------:|-------|
| ☐ | **AWS** rights: EC2, EIP, SG, IAM (roles / instance profiles), S3 catalog |
| ☐ | **Ops AWS identity** on Mac (`aws sts get-caller-identity` — **not** `pbx3-node-*`) |
| ☐ | SSH key + `.pem` (`chmod 400`) |
| ☐ | Clones of **`pbx3`** + **`pbx3cagi`** on `main` with release `.deb`s at repo root |
| ☐ | Existing fleet **org bucket** (lab: `08jzwn-pbx3`) — do **not** invent a per-node bucket |
| ☐ | **Fleet service token** (same value as Gatekeeper — see Step 6) |
| ☐ | DNS control for apex (lab: `pbx3.com`) |
| ☐ | Email for Let’s Encrypt + friendly **Name** (e.g. Sirius) |

**Lab packages (adjust when newer):** `pbx3_0.0.5-5_all.deb` · `pbx3cagi_1.0.0-18_all.deb`.

### Worksheet (fill as you go)

```bash
export AWS_DEFAULT_REGION=us-east-1
export KEY_FILE=/path/to/your.pem
export PBX3_ORG_BUCKET=08jzwn-pbx3
export LE_EMAIL=you@example.com
export SITE_NAME='Sirius'          # friendly Name — not the hostname
export INSTANCE_ID=i-…             # after launch
export PUBLIC_IP=…                 # EIP
export SHORTUID=…                  # after installer
export INSTANCE_FQDN=…             # e.g. abc12x.pbx3.com
export KSUID=…                     # globals.id
```

---

## Step 1 — Launch EC2

Ubuntu **24.04** LTS. Prefer ARM (`t4g.medium`) unless the fleet is already on x86.

**Security group (inbound):**

| Port | Why |
|------|-----|
| **22/tcp** | SSH (lock to your IP in production) |
| **80/tcp** | Let’s Encrypt HTTP-01 |
| **44300/tcp** | pbx3api HTTPS |
| SIP / RTP as needed | Phones / tests (optional on install day) |

Outbound **443** required (apt, S3, LE).

Allocate an **Elastic IP** and associate it. Save both values in the worksheet:

- **`PUBLIC_IP`** — used now for SSH / DNS / Provision edge  
- **`INSTANCE_ID`** (`i-…`) — needed later for [Step 6 onboard](#step-6--adopt-into-fleet-onboard) (`--instance-id`); not used for SSH

```bash
export PUBLIC_IP=…          # EIP after associate
export INSTANCE_ID=i-…      # EC2 instance id — keep for onboard
export SSH_HOST=ubuntu@${PUBLIC_IP}
ssh -i "$KEY_FILE" -o BatchMode=yes -o ConnectTimeout=15 "$SSH_HOST" 'hostname; uname -m'
```

---

## Step 2 — Copy packages and OS prep

**Mac:**

```bash
export PBX3_DEB=~/GiT/pbx3-master/pbx3/pbx3_0.0.5-5_all.deb
export CAGI_DEB=~/GiT/pbx3-master/pbx3cagi/pbx3cagi_1.0.0-18_all.deb
scp -i "$KEY_FILE" "$PBX3_DEB" "$CAGI_DEB" "${SSH_HOST}:/tmp/"
```

**Node:**

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y ssmtp sqlite3 curl ca-certificates git
sudo chmod +x /etc/ssmtp
cd /tmp
sudo apt install -y ./pbx3_0.0.5-5_all.deb ./pbx3cagi_1.0.0-18_all.deb
dpkg-query -W pbx3 pbx3cagi
```

`apt install` does **not** create the DB. Next step does.

---

## Step 3 — First-run installer (identity)

You do **not** type the FQDN. You supply the **apex**; the installer mints an opaque **shortuid** and writes `fqdn = {shortuid}.{apex}`.

| You provide | Installer derives / writes |
|-------------|----------------------------|
| `DOMAIN_TLD` (apex), e.g. `pbx3.com` | `shortuid` (opaque 6-char) |
| `INSTANCE_SITENAME` (friendly **Name**, e.g. Sirius) | `fqdn` = `{shortuid}.pbx3.com` |
| Admin email + password (SPA login) | `globals.id` (**KSUID** — catalog / S3 key) |

Example: apex `pbx3.com` → shortuid `7k2m9q` → FQDN **`7k2m9q.pbx3.com`**. That FQDN is what Step 5 DNS and LE use.

```bash
sudo DOMAIN_TLD=pbx3.com \
  INSTANCE_SITENAME="${SITE_NAME}" \
  PBX3_ADMIN_EMAIL=ops@example.com \
  PBX3_ADMIN_PASSWORD='choose-a-strong-password' \
  /opt/pbx3/scripts/installer.sh
```

Omit `PBX3_ADMIN_*` to answer interactive prompts on a real TTY. There is **no** factory SPA password — remember what you set.

**Do not** pass `INSTANCE_FQDN=kildare.pbx3.com` (vanity host). That path is rejected unless you intentionally break-glass with `PBX3_ALLOW_VANITY_FQDN=1`.

After install, **read** identity (this is where FQDN appears):

```bash
sqlite3 /opt/pbx3/db/sqlite.db \
  "SELECT id, shortuid, fqdn, sitename FROM globals WHERE pkey='global';"
```

Copy into the worksheet:

```bash
export KSUID=…              # id column
export SHORTUID=…           # shortuid column
export INSTANCE_FQDN=…      # fqdn column — use in Step 5 DNS/LE
```

Catalog and S3 paths use the **KSUID**, not the shortuid.

!!! warning "Do not yet"
    Do **not** set `PBX3_ORG_BUCKET` by hand — onboard owns fleet `.env`.  
    Do **not** run `reloader.sh` after a clean first install.

---

## Step 4 — Install pbx3api and prove `/up`

```bash
# Prefer clone if the node can reach GitHub; else scp a Mac checkout to /opt/pbx3api
sudo rm -rf /opt/pbx3api
sudo git clone https://github.com/aelintra/pbx3api.git /opt/pbx3api
cd /opt/pbx3api && sudo git checkout main && sudo git pull --ff-only

sudo rm -f /etc/nginx/sites-enabled/default
sudo /opt/pbx3api/scripts/installer.sh

curl -k -sS -o /dev/null -w "%{http_code}\n" https://127.0.0.1:44300/up
# expect 200
sudo systemctl is-active nginx php8.3-fpm asterisk
```

**Stop if not 200.** Onboard will not fix a broken API.

Also see [Install pbx3 and pbx3api](../installation/install-pbx3-pbx3api.md).

---

## Step 5 — DNS and Let’s Encrypt

| Name | Type | Value |
|------|------|--------|
| `globals.fqdn` (e.g. `abc12x.pbx3.com`) | **A** | EIP / `PUBLIC_IP` |

Fleet nodes do **not** publish public tenant A records.

```bash
dig +short "$INSTANCE_FQDN"   # must equal PUBLIC_IP
sudo /opt/pbx3/scripts/le-instance-bootstrap.sh "$LE_EMAIL"
# staging: sudo PBX3_LE_STAGING=1 /opt/pbx3/scripts/le-instance-bootstrap.sh "$LE_EMAIL"

curl -sS -o /dev/null -w "%{http_code}\n" "https://${INSTANCE_FQDN}:44300/up"
# prefer 200 without -k
```

SPA path: [First Let's Encrypt certificate](../tls/first-letsencrypt.md) · **Certificates → Get certificate**.

Confirm Admin SPA login with the email/password from Step 3.

---

## Step 6 — Adopt into fleet (onboard)

Run from the **ops Mac**, not as the node IAM role.

### 6a — Fleet service token

One shared secret for Gatekeeper ↔ every fleet node. Find it (do not invent a second value):

```bash
# Prefer control / gatekeeper host
ssh ubuntu@control.pbx3.com \
  "grep -E '^PBX3_FLEET_SERVICE_TOKEN=' /etc/pbx3-gatekeeper/.env | head -1"

# Or any already-onboarded sibling
ssh -i "$KEY_FILE" ubuntu@08jzwn.pbx3.com \
  "grep -E '^PBX3_FLEET_SERVICE_TOKEN=' /opt/pbx3api/.env | head -1"
```

```bash
export PBX3_FLEET_SERVICE_TOKEN='paste-value-only'
```

### 6b — Onboard script

```bash
cd ~/GiT/pbx3-master/pbx3/pbx3-directory/tools
./onboard-fleet-instance.sh \
  --instance-id "$INSTANCE_ID" \
  --ssh "$SSH_HOST" \
  --ssh-key "$KEY_FILE" \
  --region "$AWS_DEFAULT_REGION" \
  --org-bucket "$PBX3_ORG_BUCKET"
# optional: --sbc-egress-host sbc.pbx3.com
# dry-run: add --dry-run
```

**What onboard does:** node IAM + instance profile → catalog register → fleet `.env` (bucket, mode, token, egress host) → seed **Egress** trunk → genAst / runLinker / Asterisk restart → S3 smoke.

### 6c — Sign-off on the node

```bash
grep -E '^(PBX3_ORG_BUCKET|PBX3_FLEET_MODE|PBX3_FLEET_SERVICE_TOKEN)=' /opt/pbx3api/.env
sqlite3 /opt/pbx3/db/sqlite.db "SELECT pkey, active, host FROM trunks WHERE pkey='Egress';"
curl -sS http://169.254.169.254/latest/meta-data/iam/security-credentials/
cd /opt/pbx3api && sudo php artisan config:clear && sudo php artisan pbx3:fleet-preflight
```

SPA Fleet → **Instances**: refresh — new shortuid / Name should appear.

Script-only details: [Onboard a second instance](onboard-instance.md).

---

## Step 7 — SBC edge (required before Fleet Create tenant)

Onboard makes the **node** able to dial toward the SBC. It does **not** register the home on the edge.

### 7a — Provision edge (Fleet SPA)

Fleet → **Instances** → row menu → **Provision edge**.

| Field | Value |
|-------|--------|
| Backend SIP URI | **`sip:{PUBLIC_IP}:5060`** — use the **EIP / public IP**, not the FQDN |

Click **Create edge**. Catalog should show a **Setid**. That allocates a dispatcher set + Asterisk Peer (Rule 13).

!!! warning "FQDN backends"
    The SBC rejects DNS-name backends for this path. Always **`sip:IP:5060`**.

If Setid is blank after a rebuild but the edge already exists, use **Link setid** (catch-up only — does not create a new set).

### 7b — Fail2ban whitelist (manual until #5e)

SBC admin (lab: `https://sbc.pbx3.com/admin`) → **Fail2ban → Whitelist** → add this home’s public IP **`/32`**.

Without this, OPTIONS / SIP from the home can get banned and break Egress qualify.

---

## Step 8 — Optional: first tenant and smoke

1. Fleet → **Tenants** → **Create** on this home (setid must be set).  
2. Allocate / assign DIDs as needed.  
3. Desk or SIPp: inbound + outbound through the SBC.  
4. Fleet → Catalog **Reconcile** clean if you changed domains.  
5. Create a backup; confirm panel shows **local+S3** (or `pbx3:upload-backup`).

Do **not** invent node-only tenants that are missing from the catalog.

---

## Checklist

- [ ] EC2 + EIP + SG; SSH works
- [ ] `pbx3` + `pbx3cagi` installed; `installer.sh` created KSUID / opaque FQDN
- [ ] pbx3api up; `GET /up` → **200**
- [ ] DNS A + Let’s Encrypt; Admin SPA login works
- [ ] `onboard-fleet-instance.sh` (token + IAM + catalog + Egress)
- [ ] `pbx3:fleet-preflight` green; instance in Fleet picker
- [ ] **Provision edge** with `sip:{PUBLIC_IP}:5060`; Setid visible
- [ ] Fail2ban whitelist `/32` for home IP
- [ ] (Optional) tenant / DID / call smoke + S3 backup

## Failure cheat sheet

| Symptom | Likely fix |
|---------|------------|
| `/up` not 200 | pbx3api installer, nginx default site, PHP-FPM |
| LE fails | DNS A wrong; SG/Shorewall port 80 |
| Onboard AWS error | Mac using `pbx3-node-*` role — switch to ops identity |
| Preflight S3 red | IAM attach lag; empty `AWS_ACCESS_KEY_*` in `.env` |
| Fleet Create: no setid | Step 7a Provision edge |
| Fleet Create: service token | Step 6a — token must match Gatekeeper |
| Egress Unavail / home banned | Step 7b Fail2ban whitelist |
| Want **same** KSUID as a dead box | Wrong page → [Rebuild from S3](rebuild-from-s3.md) |

## Related

- [Onboard a second instance](onboard-instance.md) — adopt-only (healthy node already installed)  
- [Decommission a fleet instance](decommission-instance.md) — reverse path  
- [Rebuild from S3](rebuild-from-s3.md) — same KSUID recovery  
- [Agent-assisted onboard / rebuild](agent-assisted.md) — human+AI Mode 4  
- [Install pbx3 and pbx3api](../installation/install-pbx3-pbx3api.md) · [First Let's Encrypt](../tls/first-letsencrypt.md)
