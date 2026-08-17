# Commission a fleet instance

Join a **new home** to an existing fleet after the stack is already installed: onboard → SBC edge → optional tenant smoke.

This creates / uses a **new KSUID** from Install. It is **not** [Rebuild from S3](rebuild-from-s3.md) (same identity after EC2 loss).

| Phase | What | Where |
|-------|------|--------|
| **0** | EC2 + packages → identity → API → DNS → LE → SPA admin | **[Install pbx3 and pbx3api](../installation/install-pbx3-pbx3api.md)** (required first) |
| **1** | Fleet adopt (IAM, catalog, Egress) | This page |
| **2** | Provision edge + Fail2ban | This page |

EC2, EIP, SG (including **80** for LE and **44300** for ops/Gatekeeper), and DNS are part of Install — do not re-do them here.

!!! tip "Install not done yet?"
    Finish [Install](../installation/install-pbx3-pbx3api.md) first (trusted `/up` + SPA admin).  
    Adopt-only details also on [Onboard](onboard-instance.md).  
    Retire a home → [Decommission](decommission-instance.md).

## Before you start

| Have it? | Thing |
|:--------:|-------|
| ☐ | [Install](../installation/install-pbx3-pbx3api.md) checklist complete on this node |
| ☐ | `INSTANCE_ID` (`i-…`), `PUBLIC_IP`, SSH access + `.pem` |
| ☐ | Worksheet exports from Install: `KSUID`, `SHORTUID`, `INSTANCE_FQDN` |
| ☐ | **Ops AWS identity** on Mac (`aws sts get-caller-identity` — **not** `pbx3-node-*`) |
| ☐ | Existing fleet **org bucket** (lab: `08jzwn-pbx3`) |
| ☐ | **Fleet service token** (same as Gatekeeper — see Step 1) |

### Worksheet

```bash
export AWS_DEFAULT_REGION=us-east-1
export KEY_FILE=/path/to/your.pem
export PBX3_ORG_BUCKET=08jzwn-pbx3
export INSTANCE_ID=i-…
export PUBLIC_IP=…
export SSH_HOST=ubuntu@${PUBLIC_IP}
# or: export SSH_HOST=ubuntu@virginia1.pbx3.com

export KSUID=…
export SHORTUID=…
export INSTANCE_FQDN=…
```

---

## Step 1 — Adopt into fleet (onboard)

Run from the **ops Mac**, not as the node IAM role.

### 1a — Fleet service token

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

### 1b — Onboard script

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

### 1c — Sign-off

**On the node:**

```bash
grep -E '^(PBX3_ORG_BUCKET|PBX3_FLEET_MODE|PBX3_FLEET_SERVICE_TOKEN)=' /opt/pbx3api/.env
sqlite3 /opt/pbx3/db/sqlite.db "SELECT pkey, active, host FROM trunks WHERE pkey='Egress';"
curl -sS http://169.254.169.254/latest/meta-data/iam/security-credentials/
cd /opt/pbx3api && sudo php artisan config:clear && sudo php artisan pbx3:fleet-preflight
```

Fleet SPA → **Instances** → refresh — new shortuid / Name should appear.

---

## Step 2 — SBC edge (required before Fleet Create tenant)

Onboard makes the **node** able to dial toward the SBC. It does **not** register the home on the edge.

### 2a — Provision edge (Fleet SPA)

Fleet → **Instances** → row menu → **Provision edge**.

| Field | Value |
|-------|--------|
| Backend SIP URI | **`sip:{PUBLIC_IP}:5060`** — EIP / public IP, **not** the FQDN |

Click **Create edge**. Catalog should show a **Setid**.

!!! warning "FQDN backends"
    The SBC rejects DNS-name backends here. Always **`sip:IP:5060`**.

If Setid is blank after a rebuild but the edge already exists, use **Link setid** (catch-up only).

### 2b — Fail2ban whitelist (manual until #5e)

SBC admin (lab: `https://sbc.pbx3.com/admin`) → **Fail2ban → Whitelist** → add this home’s public IP **`/32`**.

---

## Step 3 — Optional: first tenant and smoke

1. Fleet → **Tenants** → **Create** on this home (setid required).  
2. Allocate / assign DIDs.  
3. Inbound + outbound through the SBC.  
4. Catalog **Reconcile** if domains changed.  
5. Backup → panel shows **local+S3**.

Do **not** invent node-only tenants missing from the catalog.

---

## Checklist

- [ ] [Install](../installation/install-pbx3-pbx3api.md) complete
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
| Fleet Create: no setid | Step 2a Provision edge |
| Fleet Create: service token | Step 1a — token must match Gatekeeper |
| Egress Unavail / home banned | Step 2b Fail2ban whitelist |
| Want **same** KSUID as a dead box | [Rebuild from S3](rebuild-from-s3.md) |

## Related

- [Install pbx3 and pbx3api](../installation/install-pbx3-pbx3api.md) — required before this page  
- [Onboard](onboard-instance.md) — same adopt script  
- [Decommission](decommission-instance.md) · [Rebuild from S3](rebuild-from-s3.md) · [Agent-assisted](agent-assisted.md)
