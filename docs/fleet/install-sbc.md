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

## Install order

1. Install **edge** (`pbx3sbc`) — OpenSIPS, MariaDB, firewall helpers  
2. Install **admin** (`pbx3sbc-admin`) — PHP/nginx/Filament (**`--server-name` required**)  
3. Smoke: services + admin login  
4. HTTPS: installer `--letsencrypt` **or** Filament **Certificates** (SPA-like)  
5. Optional: seed peers/domains, or [restore a backup](sbc-backup-restore.md)

---

## 1. Clone and install the edge

```bash
sudo apt-get update && sudo apt-get install -y git curl
cd ~
git clone https://github.com/aelintra/pbx3sbc.git
cd pbx3sbc
sudo ./install.sh --advertised-ip <PUBLIC_EIP_OR_VIP> --db-password '<DB_PASSWORD>'
```

When prompted **Reinitialize database? (y/N):** answer **`y`** on a fresh host.

Check: `systemctl is-active opensips mariadb` → both `active`.

---

## 2. Clone and install the admin

**`--server-name <PUBLIC_FQDN>` is required** (nginx + `APP_URL`). Do not use the EC2 internal hostname.

```bash
cd ~
git clone https://github.com/aelintra/pbx3sbc-admin.git
cd pbx3sbc-admin

sudo ./install.sh \
  --server-name <PUBLIC_FQDN> \
  --db-host localhost \
  --db-name opensips \
  --db-user opensips \
  --db-password '<DB_PASSWORD>' \
  --opensips-mi-url http://127.0.0.1:8888/mi \
  --admin-name Admin \
  --admin-email '<ADMIN_EMAIL>' \
  --admin-password '<ADMIN_PASSWORD>'
```

Optional HTTPS in the same pass (DNS + SG 80/443 ready):

```bash
sudo ./install.sh \
  --server-name <PUBLIC_FQDN> \
  --letsencrypt --email '<LE_EMAIL>' \
  ... # same DB / admin flags
```

Installer also applies Fail2ban/Certificates **sudoers** when `pbx3sbc` is present next to this clone.

### Certificates panel (SPA kinship)

Filament **Certificates** mirrors the instance SPA:

| Section | Behaviour |
|---------|-----------|
| **Let's Encrypt** | Hostname from `APP_URL` (readonly); email + **Get certificate**; when configured: status + **Renew now** |
| **Purchased certificate** | Upload fullchain + key; Install / Remove |

Edge vs SPA: **no** multi-tenant SAN “Sync with tenant list” (single admin FQDN).

---

## 3. HTTPS after HTTP install

Filament → **Certificates** → **Get certificate**, or:

```bash
sudo ~/pbx3sbc-admin/scripts/le-admin-cert.sh setup \
  <PUBLIC_FQDN> <LE_EMAIL> /home/ubuntu/pbx3sbc-admin/public
```

Details: `pbx3sbc/workingdocs/LE_HTTPS_SBC_ADMIN.md`. SIP TLS out of scope.

---

## Active–passive HA pair (lab shape)

| Member | Owns EIP? | `advertised-ip` | `--server-name` |
|--------|-----------|-----------------|-----------------|
| Active | Yes | **EIP** | Shared FQDN |
| Standby | No | **Same EIP** | Same FQDN |

Promote = fence old active → reassociate EIP. Issue LE on whoever currently owns the EIP.

## Related

- [SBC backup and restore](sbc-backup-restore.md)
- [Fleet overview](overview.md)
