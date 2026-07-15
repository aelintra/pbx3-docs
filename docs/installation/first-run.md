# First-run installer

## When to run `installer.sh`

| Situation | Run? |
|-----------|------|
| Brand-new node after package install | **Yes** — once |
| Routine package upgrade | **No** — postinst handles identity normalize |
| Empty identity fields after git scripts | Prefer `normalize-globals-identity.sh` |
| Existing `sqlite.db` present | Re-run **preserves** DB / FQDN / shortuid unless you force apply |

Force identity apply (rare):

```bash
sudo PBX3_APPLY_INSTANCE_IDENTITY=1 INSTANCE_FQDN=node.example.com /opt/pbx3/scripts/installer.sh
```

## What **not** to run casually

| Script | Risk |
|--------|------|
| `/opt/pbx3/scripts/reloader.sh` | Full schema reload — **not** routine; never after restore |
| Random hand-edits to `globals.id` | Breaks S3 prefixes and catalog rows |

Normalize empty UID fields without wipe:

```bash
sudo /opt/pbx3/scripts/normalize-globals-identity.sh
```

See also [Restore from backup](restore-from-backup.md).
