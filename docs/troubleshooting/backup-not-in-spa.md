# Backup not visible in SPA

| Symptom | Fix |
|---------|-----|
| No S3 rows under instance | Create backup while node healthy; confirm upload; `aws s3 ls s3://ORG/instances/{KSUID}/backups/` |
| Local gone, S3 present | Expected (local FIFO ~9); fetch from S3 for rebuild |
| Upload fail | Instance profile attached; remove empty AWS key env lines; policy KSUID = `globals.id`; egress 443 |
| Lifecycle script denied on node | Run lifecycle from **Mac ops** identity |

See [Backups in the org bucket](../fleet/backups-org-bucket.md).
