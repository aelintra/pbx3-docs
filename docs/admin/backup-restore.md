# Backup and restore

Route: `/backup` (Snapshots may be a separate `/snapshots` panel)

## Create

1. Open **Backup**.
2. **Create** → local zip under `/opt/pbx3/bkup/pbx3bak.{epoch}.zip`.
3. Fleet with upload enabled: async put to org S3 under `instances/{KSUID}/backups/`.

## Index

- Local rows + S3 rows (UTC + Archive ID).
- Local retention: newest **~9** (FIFO). S3 lifecycle often **~30d** on tagged backups — pruning local does **not** delete S3.

## Restore

See [Restore from backup](../installation/restore-from-backup.md). Lab: rehearse once.

## Snapshots

DB-oriented create/restore/delete (Commit may also snapshot depending on release). Prefer Backup zips for full appliance recovery.
