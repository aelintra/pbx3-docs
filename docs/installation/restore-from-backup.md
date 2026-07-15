# Restore from backup

## Prefer SPA when possible

**Backup** panel → choose archive → Restore. Lab: rehearse once before you need it.

## Full zip on the node

```bash
sudo /opt/pbx3/scripts/restore-backup-zip.sh --full /opt/pbx3/bkup/pbx3bak.*.zip
```

## After restore (operator checklist)

1. Confirm `globals` identity matches **this** node (same-KSUID rebuild: already correct; cross-node: patch carefully).
2. Default tenant `fqdn` and DNS for tenant FQDNs point here.
3. Certificates → **Sync with tenant list** (not Renew alone).
4. Merge help seeds if UI hints are empty:

```bash
sudo sqlite3 /opt/pbx3/db/sqlite.db < /opt/pbx3/db/db_sql/sqlite_message.sql
```

5. **Never** run `reloader.sh` after restore.
6. SPA → **Commit** if generators need a pass.

## Fleet: restore from org bucket

See [Rebuild a fleet node from S3](../fleet/rebuild-from-s3.md) (`fetch-latest-instance-backup.sh`).
