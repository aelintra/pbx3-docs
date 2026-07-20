# Backups in the org bucket

**Instance (PBX) backups** — nodes upload under `instances/{ksuid}/…`. For the **SBC edge**, see [SBC backup and restore](sbc-backup-restore.md).

## Layout

```text
s3://{org}/instances/{globals.id}/backups/{UTC_stamp}/
  backup.zip
  manifest.json
```

Lab org: `08jzwn-pbx3`.

## Create / upload

- SPA **Backup** → Create (async upload when fleet enabled), or
- `php artisan pbx3:backup-run --trigger=manual`
- `sudo php artisan pbx3:upload-backup basename.zip`

## Retention

- Local: ~9 newest zips
- S3: lifecycle on tagged `class=backup` (often 30d) under **`instances/`** and **`sbc/`** — apply from **Mac ops**, not the node role:

```bash
./pbx3-directory/tools/apply-backup-lifecycle-rule.sh 08jzwn-pbx3 30
```

Rebuild recovery point = last successful **S3** upload, not “whatever was on disk at crash time”.
