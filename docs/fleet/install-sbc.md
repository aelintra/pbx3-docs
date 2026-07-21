# Install SBC edge (pbx3sbc + admin)

Greenfield install of the **SIP edge** (`pbx3sbc` / OpenSIPS + MariaDB) and the **Filament admin** (`pbx3sbc-admin`) on **Ubuntu 24.04 amd64**.

This is **not** [SBC backup and restore](sbc-backup-restore.md) (cold DR) and **not** an HA promote drill — but the same install is the baseline for an active–passive pair (shared EIP / VIP as `advertised_address`).

!!! note "Architecture"
    OpenSIPS packages from `apt.opensips.org` on Ubuntu 24.04 are **amd64 (x86_64)** today. Use `t3.*` (or other x86_64), **not** `t4g`, unless you build OpenSIPS yourself.

## Prerequisites

| Need | Notes |
|------|--------|
| Host | Fresh **Ubuntu 24.04** x86_64, SSH as a sudoer (lab: `ubuntu`) |
| Disk | Prefer **≥ 16 GB** root; ~8 GB works but is tight after apt + PHP |
| Network | Outbound HTTPS for apt + GitHub clones |
| DNS (admin HTTPS) | A record for the public FQDN → **stable** public IPv4 (EIP) |
| Security group / firewall | **UDP/TCP 5060** (SIP), **TCP 22**, and for admin/LE: **TCP 80 + 443** from the internet (validators need world `80/443`) |

Optional later: RTP ports if media terminates on the SBC; Fail2ban / log-ship / backup cron — see related pages after first smoke.

## Install order

1. Install **edge** (`pbx3sbc`) — OpenSIPS, MariaDB, firewall helpers  
2. Install **admin** (`pbx3sbc-admin`) — PHP/nginx/Filament against the same MariaDB  
3. Smoke: services + admin login  
4. Optional: [Let’s Encrypt HTTPS](#optional-https-lets-encrypt)  
5. Optional: seed peers/domains, or [restore a backup](sbc-backup-restore.md)

---

## 1. Clone and install the edge

On the host:

```bash
sudo apt-get update
sudo apt-get install -y git curl

cd ~
git clone https://github.com/aelintra/pbx3sbc.git
cd pbx3sbc

sudo ./install.sh \
  --advertised-ip <PUBLIC_EIP_OR_VIP> \
  --db-password '<DB_PASSWORD>'
```

| Flag | Meaning |
|------|---------|
| `--advertised-ip` | Address phones / carriers / fleet `Egress` see in SIP. On **AWS**, use the **Elastic IP** (or future VIP), **not** the instance private IP. |
| `--db-password` | MariaDB password for user/db `opensips` (stored under `/etc/opensips/.mysql_credentials`). |
| `--preferlan` | Lab-only: prefer a private LAN IP for `advertised_address`. **Do not** use on public EC2 HA members. |

When prompted **Reinitialize database? (y/N):**

- **Fresh host:** answer **`y`**
- **Reinstall keeping data:** answer **`N`**

Check:

```bash
systemctl is-active opensips mariadb
# → active / active

grep advertised_address /etc/opensips/opensips.cfg
# → advertised_address="<PUBLIC_EIP_OR_VIP>"
```

---

## 2. Clone and install the admin

Use the **same** DB password as the edge install. Pass a **public FQDN** for nginx `server_name` (not the EC2 internal hostname).

```bash
cd ~
git clone https://github.com/aelintra/pbx3sbc-admin.git
cd pbx3sbc-admin

sudo ./install.sh \
  --db-host localhost \
  --db-name opensips \
  --db-user opensips \
  --db-password '<DB_PASSWORD>' \
  --server-name <PUBLIC_FQDN> \
  --opensips-mi-url http://127.0.0.1:8888/mi \
  --admin-name Admin \
  --admin-email '<ADMIN_EMAIL>' \
  --admin-password '<ADMIN_PASSWORD>'
```

If you omit `--admin-*`, the installer creates a user with a generated password and prints it once — save it.

!!! note "Shared DB with OpenSIPS"
    Edge `init-database.sh` pre-creates Laravel `sessions` / `cache` tables. Admin migrations are idempotent (`Schema::hasTable`) so a greenfield `php artisan migrate` does not fail with “Table 'sessions' already exists”. Older admin checkouts without that fix: pull latest `pbx3sbc-admin` or re-run migrate after updating those migration files.

Check:

```bash
systemctl is-active nginx
curl -s -o /dev/null -w "%{http_code}\n" http://127.0.0.1/admin/login
# → 200 (or 302 to login)
```

From your laptop (SG must allow your IP on **80**):

`http://<PUBLIC_FQDN>/admin` or `http://<PUBLIC_IP>/admin`

---

## 3. Optional: HTTPS (Let’s Encrypt)

After DNS for `<PUBLIC_FQDN>` points at the box’s public IPv4:

1. Open **TCP 80 and 443** on the **EC2 security group** to `0.0.0.0/0` (office-IP-only SGs break ACME).
2. Follow the lab runbook in the `pbx3sbc` repo: `workingdocs/LE_HTTPS_SBC_ADMIN.md`
3. Use nginx template `pbx3sbc-admin/deploy/nginx-pbx3sbc-admin.conf`
4. Set `APP_URL=https://<PUBLIC_FQDN>` in `/home/ubuntu/pbx3sbc-admin/.env` and `php artisan config:clear`
5. Rotate / confirm the Filament admin password after HTTPS is live

SIP TLS (`5061`) is **out of scope** for that runbook.

---

## Active–passive HA pair (lab shape)

Requirements: fleet docs **SBC HA failover** (VIP/EIP + warm standby). Install summary for a throwaway pair:

| Member | Public IP | Owns EIP? | `advertised-ip` | nginx `--server-name` |
|--------|-----------|-----------|-----------------|------------------------|
| Active (e.g. sbcFO1) | EIP | Yes | **EIP** | Shared FQDN (e.g. `sbcfo.pbx3.com`) |
| Standby (e.g. sbcFO2) | Ephemeral | No | **Same EIP** (not its own public IP) | Same FQDN |

- Each member has **local MariaDB** (no shared DB).
- DNS A for the shared FQDN → EIP.
- **Promote** = fence old active → reassociate EIP onto standby. Soft state (registrations / mid-calls) is not moved.
- Do **not** point production carriers / Magrathea VIP at the pair until a timed promote drill passes.

Warm sync (catalog project + edge-authored copy cadence) and the promote runbook are separate follow-ons after both members install cleanly.

---

## Post-install ops (common)

| Task | Where |
|------|--------|
| Fail2ban / sudoers for admin panels | `pbx3sbc/scripts/setup-admin-panel-sudoers.sh` |
| Cold backup / restore | [SBC backup and restore](sbc-backup-restore.md) |
| Data / log aging | [SBC data retention](sbc-data-retention.md) |
| Log ship to org bucket | `pbx3sbc/config/log-ship.env.example` → `/etc/pbx3sbc/log-ship.env` |

---

## Smoke checklist

- [ ] `opensips`, `mariadb`, `nginx` are `active`
- [ ] `advertised_address` is the **stable** public EIP/VIP
- [ ] Filament login works over HTTP (then HTTPS if enabled)
- [ ] `opensips-cli -x mi ps` (or equivalent MI) responds on `127.0.0.1:8888`
- [ ] SG allows SIP **5060** from peers you intend to test

---

## Related

- [SBC backup and restore](sbc-backup-restore.md) — cold DR (install both products, then restore zip)
- [Fleet overview](overview.md)
- Edge HA requirements (source): `pbx3/pbx3-directory/docs/SBC_HA_FAILOVER_REQUIREMENTS.md`
