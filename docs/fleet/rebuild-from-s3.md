# Rebuild a fleet node from S3

Same **KSUID** and **FQDN** — recover appliance from the last good backup in the org bucket.

## Hard stop

Confirm S3 has backups **before** destroying the old VM:

```bash
aws s3 ls s3://08jzwn-pbx3/instances/{KSUID}/backups/
```

## Order

1. Launch empty Ubuntu 24.04 → install `pbx3` + `pbx3api` (installers). **Do not** attach IAM / set org bucket yet.
2. Mac: fetch latest zip:

```bash
./fetch-latest-instance-backup.sh --instance-id {KSUID} --output-dir /tmp
```

3. `scp` zip to node → `restore-backup-zip.sh --full` → merge `sqlite_message.sql`. **No** `reloader.sh`.
4. `./onboard-fleet-instance.sh` (IAM + `.env` + smoke).
5. DNS A for node + tenants → new IP → Cert **Sync** → Commit → `pbx3:fleet-preflight` all green.

## Lab golden hints

| Field | Example |
|-------|---------|
| KSUID | `3DmAsxePTWQZgynBYXE8obIRqEE` |
| FQDN | `08jzwn.pbx3.com` |
| Bucket | `08jzwn-pbx3` |

Prefer blue/green (bring new IP up, test, then DNS) over in-place surprise cutover.

Human+AI path: [Agent-assisted onboard / rebuild](agent-assisted.md).
