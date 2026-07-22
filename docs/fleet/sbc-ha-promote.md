# SBC HA — cast-iron warm sync and EIP promote

Active–passive edge HA. **Follow this procedure in order. Do not improvise.**  
Requirements: `pbx3-directory/docs/SBC_HA_FAILOVER_REQUIREMENTS.md`.

!!! abstract "Operator rule"
    If the pair was installed and kept warm per Phases A–B, a promote is a **checklist**, not a design exercise. Fill in the blanks from the pair’s runbook card (instance ids, EIP alloc, FQDN, LE email, SSH key). Tick every **Done when** box before stopping.

!!! note "Managed vs auto"
    This page is the **human / managed** path. Fleet may later choose **`auto`** (control promotes). If control is dark, still use this page — you are not stuck. Spec: `SBC_HA_FAILOVER_REQUIREMENTS.md`.

**Promote is not finished** until **both**:

1. SIP answers on the EIP (OPTIONS / REGISTER)  
2. **`https://<FQDN>/admin/login`** shows the Filament login page  

Moving the EIP does **not** move Let’s Encrypt files — Phase D is mandatory.

This is **not** [SBC backup and restore](sbc-backup-restore.md) (cold DR).

!!! warning "Lab vs live carriers"
    Do **not** point Magrathea / live `sbc.pbx3.com` at a throwaway pair until this checklist has passed and carriers allowlist the stable address.

---

## Pair card (fill once; keep with the runbook)

Copy these values when the pair is built. Incident responders use **only** this card + the phases below.

| Blank | Lab example | This pair |
|-------|-------------|-----------|
| FQDN | `sbcfo.pbx3.com` | |
| EIP | `98.82.58.59` | |
| EIP allocation id | `eipalloc-020e72437124c600e` | |
| Member A instance id | `i-05b30224300cc8812` | |
| Member B instance id | `i-00f85b1c3f18c434e` | |
| Who is active *now*? | B after 2026-07-21 drill | |
| Who is standby *now*? | A | |
| SSH key | `opensips.pem` / `ubuntu@` | |
| AWS region | `us-east-1` | |
| LE email | (ACME contact) | |
| Admin path | `/admin/login` | |

Greenfield install: [Install SBC edge](install-sbc.md).

---

## Phase A — Pair install

| Member | Owns EIP at install? | `advertised-ip` | `--server-name` | `--letsencrypt` |
|--------|----------------------|-----------------|-----------------|-----------------|
| **Active** | Yes | **EIP** | Shared FQDN | **Yes** (`--email`) |
| **Standby** | No | **Same EIP** | Same FQDN | **No** |

**Done when**

- [ ] Both: `systemctl is-active opensips` → `active`
- [ ] Both: `advertised_address` = the **EIP**
- [ ] Both: same FQDN / nginx `server_name`
- [ ] Active: `https://<FQDN>/admin/login` → login page
- [ ] Standby: `http://<standby-public-ip>/admin/login` → login page
- [ ] Pair card filled

---

## Phase B — Warm sync (keep standby ready)

**Preferred:** Fleet → **Edge HA → Sync now** (control: active `POST /api/fleet/backup` → S3 → standby `POST /api/fleet/warm-pull` with `--db-only`). Daily backstop: `pbx3-edge-warm-sync.timer` on control (`05:30` UTC).

**When:** after any edge-authored change on the active, or rely on the daily timer. Do **not** treat Sync now as failover — that is Phase C.

**Manual fallback** (scp):

```bash
# On ACTIVE
sudo /home/ubuntu/pbx3sbc/scripts/backup-sbc.sh --trigger manual
sudo cp /var/lib/pbx3sbc/bkup/sbcbak.EPOCH.zip /tmp/
sudo chown ubuntu:ubuntu /tmp/sbcbak.EPOCH.zip
# scp zip → standby:/tmp/

# On STANDBY
sudo /home/ubuntu/pbx3sbc/scripts/restore-sbc-backup.sh --db-only --yes --restart /tmp/sbcbak.EPOCH.zip
```

**Done when**

- [ ] Fleet card shows last warm sync stamp (or manual checklist below)
- [ ] Standby OpenSIPS `active`
- [ ] Standby `advertised_address` still = EIP
- [ ] Standby `APP_URL` unchanged by restore

---

## Phase C — Promote SIP (incident)

Record start time. Target ≤ 20 minutes to Phase C **Done when**.

!!! danger "Expected"
    After step 2, `https://<FQDN>/…` may fail until Phase D. That is normal. Do not roll the EIP back for that reason.

### C1 — Fence old active (if reachable)

```bash
# SSH to current active (FQDN or EIP). If SSH fails, skip to C2.
sudo systemctl stop opensips
```

- [ ] OpenSIPS stopped on old active **or** host unreachable (skip)

### C2 — Move EIP onto standby (pick ONE method)

**Method CLI** (ops laptop):

```bash
aws ec2 associate-address --region <REGION> \
  --instance-id <STANDBY_INSTANCE_ID> \
  --allocation-id <EIP_ALLOCATION_ID> \
  --allow-reassociation
```

**Method Console** (no CLI, no gatekeeper):

1. AWS Console → **EC2** → **Elastic IPs**
2. Select the edge EIP from the pair card
3. **Actions** → **Associate Elastic IP address**
4. Instance = **standby** instance id from the pair card
5. Allow reassociation if prompted → Associate

- [ ] EIP shows associated to standby instance id

### C3 — Verify SIP

```bash
# SSH to EIP (now standby)
systemctl is-active opensips
# Expect: active
# SIP OPTIONS to EIP:5060 → SIP/2.0 200 OK
```

**Done when**

- [ ] EIP on standby instance id
- [ ] OpenSIPS `active` on that host
- [ ] OPTIONS (or REGISTER) **200** on EIP:5060
- [ ] Wall-clock noted
- [ ] Pair card updated (who is active / standby *now*)

---

## Phase D — Promote admin HTTPS (mandatory)

Do **not** stop after Phase C.

```bash
# SSH to NEW active (EIP)
sudo /home/ubuntu/pbx3sbc-admin/scripts/le-admin-cert.sh setup \
  <FQDN> <LE_EMAIL> /home/ubuntu/pbx3sbc-admin/public

sudo sed -i 's|^APP_URL=.*|APP_URL=https://<FQDN>|' /home/ubuntu/pbx3sbc-admin/.env
cd /home/ubuntu/pbx3sbc-admin && sudo -u www-data php artisan config:clear
```

**Done when**

- [ ] `le-admin-cert.sh status <FQDN>` → `"configured":true`
- [ ] `https://<FQDN>/admin/login` → **200** (login page)
- [ ] `APP_URL=https://<FQDN>`

**Promote complete.** Stop.

---

## Phase E — Fail-back (planned drill only)

Same checklist, other direction:

1. Phase B — warm the member that will become standby  
2. Phase C — fence current active → EIP to the other instance  
3. Phase D — LE on whoever newly owns the EIP  

---

## Control plane dark?

Same Phases **C → D**. Gatekeeper is not on this checklist. Method Console in C2 is enough for the address move if the standby was kept warm (Phase B).

---

## Lab reference (2026-07-21)

| Role | EC2 | Notes |
|------|-----|--------|
| FO1 | `i-05b30224300cc8812` | Member A — **active** after auto-promote drill 2026-07-21 |
| FO2 | `i-00f85b1c3f18c434e` | Member B — standby |
| EIP | `eipalloc-020e72437124c600e` / `98.82.58.59` | `sbcfo.pbx3.com` |
| SSH | `opensips.pem` | `ubuntu@…` |
| Control | `control.pbx3.com` | Edge probe timer; Fleet → **Edge HA** |

## Related

- [Install SBC edge](install-sbc.md)
- [SBC backup and restore](sbc-backup-restore.md)
- Requirements: `SBC_HA_FAILOVER_REQUIREMENTS.md`
