# SBC data retention (MySQL aging)

Operators keep a bounded window of **edge** security events and CDR on the SBC MariaDB. Daily cron deletes older rows. This is **not** instance Asterisk CDR, and **not** SBC backup/restore.

## What is kept

| Data | Admin panel | Default local days |
|------|-------------|-------------------|
| Door-knock attempts | Logs → Door Knock | **30** |
| Failed registrations | Logs → Failed Registrations | **30** (same job) |
| Edge CDR (`acc`) | Logs → CDR | **90** (edge ops only) |

Text logs and SIP pcap are rotated separately (already shipped).

## Knobs

In **SBC Admin → Logs → Data retention**:

- Set retention days and batch size (saved to an override file on the host).
- View **last purge** status (written by cron/CLI).

Saving knobs does **not** delete rows. Destructive purge is **cron / artisan only**.

Env defaults (if you never save in the UI):

```bash
PBX3_SBC_SECURITY_EVENTS_LOCAL_DAYS=30
PBX3_SBC_ACC_LOCAL_DAYS=90
PBX3_SBC_PURGE_BATCH_SIZE=1000
```

## Schedule

On the SBC host (`/etc/cron.d/pbx3sbc-retention`):

- **06:15** — security events
- **06:20** — `acc`

## Manual / dry-run

```bash
cd /home/ubuntu/pbx3sbc-admin   # lab path; adjust if needed
php artisan pbx3sbc:purge-security-events --dry-run
php artisan pbx3sbc:purge-acc --dry-run
```

## Related

- [SBC backup and restore](sbc-backup-restore.md) — MariaDB dump → S3 → cold restore (production gate).
- Product call history remains on the **instance** (Asterisk CDR), not SBC `acc`.
