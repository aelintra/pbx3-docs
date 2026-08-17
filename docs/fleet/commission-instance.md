# Commission a fleet instance

Bring a **new home** into an existing fleet: launch EC2 → **install the stack** → onboard → SBC edge.

This creates a **new KSUID**. It is **not** [Rebuild from S3](rebuild-from-s3.md) (same identity after EC2 loss).

| Phase | What | Where the detail lives |
|-------|------|------------------------|
| **1** | Launch EC2 + EIP + SG | This page |
| **2** | Packages → identity → API → DNS → LE → admin SPA | **[Install pbx3 and pbx3api](../installation/install-pbx3-pbx3api.md)** (do that page end-to-end) |
| **3** | Fleet adopt (IAM, catalog, Egress) | This page (onboard) |
| **4** | Provision edge + Fail2ban | This page |

!!! tip "Already finished Install?"
    Skip to [Step 3 — Adopt into fleet](#step-3--adopt-into-fleet-onboard).  
    Stack-only / solo stop → [Install](../installation/install-pbx3-pbx3api.md).  
    Retire a home → [Decommission](decommission-instance.md).

## Before you start

| Have it? | Thing |
|:--------:|-------|
| ☐ | Everything on the [Install](../installation/install-pbx3-pbx3api.md) “Before you start” list (clones, debs, DNS, LE email, admin password) |
| ☐ | **AWS** rights: EC2, EIP, SG, IAM, S3 catalog |
| ☐ | **Ops AWS identity** on Mac (`aws sts get-caller-identity` — **not** `pbx3-node-*`) |
| ☐ | Existing fleet **org bucket** (lab: `08jzwn-pbx3`) — do **not** invent a per-node bucket |
| ☐ | **Fleet service token** (same value as Gatekeeper — see Step 3) |

### Worksheet

Put comments on their **own** lines (zsh). Use **straight** quotes only.

```bash
export AWS_DEFAULT_REGION=us-east-1
export KEY_FILE=/path/to/your.pem
export PBX3_ORG_BUCKET=08jzwn-pbx3

# After launch:
# export INSTANCE_ID=i-…
# export PUBLIC_IP=…
# export SSH_HOST=ubuntu@…

# After Install (from node sqlite / installer banner):
# export KSUID=…
# export SHORTUID=…
# export INSTANCE_FQDN=…
```

---

## Step 1 — Launch EC2

Ubuntu **24.04** LTS. Prefer ARM (`t4g.medium`) unless the fleet is already on x86.

**Security group (inbound):**

| Port | Why |
|------|-----|
| **22/tcp** | SSH |
| **80/tcp** | Let’s Encrypt (`0.0.0.0/0` is fine) |
| **44300/tcp** | API/SPA — allow **ops Mac `/32`** and **Gatekeeper `/32`** (port 80 alone is not enough) |
| SIP / RTP as needed | Optional on install day |

Outbound **443** required (apt, S3, LE).

Allocate an **Elastic IP** and associate it:

- **`PUBLIC_IP`** — SSH, DNS, Provision edge  
- **`INSTANCE_ID`** (`i-…`) — required later for onboard `--instance-id`

```bash
export PUBLIC_IP=…
export INSTANCE_ID=i-…
export SSH_HOST=ubuntu@${PUBLIC_IP}
ssh -i "$KEY_FILE" -o BatchMode=yes -o ConnectTimeout=15 "$SSH_HOST" 'hostname; uname -m'
```

---

## Step 2 — Install the stack (full Install guide)

Do **[Install pbx3 and pbx3api](../installation/install-pbx3-pbx3api.md)** completely on this host:

copy debs → `installer.sh` → pbx3api → prove `/up` on **127.0.0.1** → DNS **A** for opaque `globals.fqdn` → Let’s Encrypt → SPA admin user → trusted `/up` from the **Mac**.

Do **not** set `PBX3_ORG_BUCKET` by hand during Install — onboard owns fleet `.env`.

When Install’s checklist is done, copy identity into this worksheet (`KSUID`, `SHORTUID`, `INSTANCE_FQDN`) and continue here.

---

## Step 3 — Adopt into fleet (onboard)

Run from the **ops Mac**, not as the node IAM role. Same script as [Onboard](onboard-instance.md); this is the new-home path’s adopt step.

### 3a — Fleet service token

One shared secret for Gatekeeper ↔ every fleet node. Do not invent a second value:

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

### 3b — Onboard script

```bash
cd ~/GiT/pbx3-master/pbx3/pbx3-directory/tools
./onboard-fleet-instance.sh \
  --instance-id "$INSTANCE_ID" \
  --ssh "$SSH_HOST" \
  --ssh-key "$KEY_FILE" \
  --region "$AWS_DEFAULT_REGION" \
  --org-bucket "$PBX3_ORG_BUCKET"
```

**What it does:** node IAM + instance profile → catalog register → fleet `.env` → seed **Egress** → genAst / runLinker / Asterisk restart → S3 smoke.

### 3c — Sign-off

**On the node:**

```bash
grep -E '^(PBX3_ORG_BUCKET|PBX3_FLEET_MODE|PBX3_FLEET_SERVICE_TOKEN)=' /opt/pbx3api/.env
sqlite3 /opt/pbx3/db/sqlite.db "SELECT pkey, active, host FROM trunks WHERE pkey='Egress';"
curl -sS http://169.254.169.254/latest/meta-data/iam/security-credentials/
cd /opt/pbx3api && sudo php artisan config:clear && sudo php artisan pbx3:fleet-preflight
```

Fleet SPA → **Instances** → refresh — new shortuid / Name should appear.

---

## Step 4 — SBC edge (required before Fleet Create tenant)

Onboard makes the **node** able to dial toward the SBC. It does **not** register the home on the edge.

### 4a — Provision edge (Fleet SPA)

Fleet → **Instances** → row menu → **Provision edge**.

| Field | Value |
|-------|--------|
| Backend SIP URI | **`sip:{PUBLIC_IP}:5060`** — EIP / public IP, **not** the FQDN |

Click **Create edge**. Catalog should show a **Setid**.

!!! warning "FQDN backends"
    The SBC rejects DNS-name backends here. Always **`sip:IP:5060`**.

If Setid is blank after a rebuild but the edge already exists, use **Link setid** (catch-up only).

### 4b — Fail2ban whitelist (manual until #5e)

SBC admin (lab: `https://sbc.pbx3.com/admin`) → **Fail2ban → Whitelist** → add this home’s public IP **`/32`**.

---

## Step 5 — Optional: first tenant and smoke

1. Fleet → **Tenants** → **Create** on this home (setid required).  
2. Allocate / assign DIDs.  
3. Inbound + outbound through the SBC.  
4. Catalog **Reconcile** if domains changed.  
5. Backup → panel shows **local+S3**.

Do **not** invent node-only tenants missing from the catalog.

---

## Checklist

- [ ] EC2 + EIP + SG (44300 for ops + Gatekeeper)
- [ ] [Install](../installation/install-pbx3-pbx3api.md) checklist complete (opaque FQDN, LE, SPA admin)
- [ ] `onboard-fleet-instance.sh` (token + IAM + catalog + Egress)
- [ ] `pbx3:fleet-preflight` green; instance in Fleet picker
- [ ] **Provision edge** `sip:{PUBLIC_IP}:5060`; Setid visible
- [ ] Fail2ban whitelist `/32` for home IP
- [ ] (Optional) tenant / DID / call smoke + S3 backup

## Failure cheat sheet

| Symptom | Likely fix |
|---------|------------|
| Stack / `/up` / LE / hairpin / empty users | [Install failure cheat sheet](../installation/install-pbx3-pbx3api.md#failure-cheat-sheet) |
| Onboard AWS error | Mac using `pbx3-node-*` role — switch to ops identity |
| Preflight S3 red | IAM attach lag; empty `AWS_ACCESS_KEY_*` in `.env` |
| Fleet Create: no setid | Step 4a Provision edge |
| Fleet Create: service token | Step 3a — token must match Gatekeeper |
| Egress Unavail / home banned | Step 4b Fail2ban whitelist |
| Want **same** KSUID as a dead box | [Rebuild from S3](rebuild-from-s3.md) |

## Related

- [Install pbx3 and pbx3api](../installation/install-pbx3-pbx3api.md) — stack procedure (Phase 2)  
- [Onboard](onboard-instance.md) — adopt-only when Install is already done  
- [Decommission](decommission-instance.md) · [Rebuild from S3](rebuild-from-s3.md) · [Agent-assisted](agent-assisted.md)
