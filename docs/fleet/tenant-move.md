# Tenant move

High-level order (panel Jobs / Move wizard may wrap this):

1. Prep destination capacity / trunk mapping.
2. Export on source → import on destination → **Commit** → test on dest **before** DNS.
3. DNS A for tenant FQDN → destination.
4. Cert **Sync** on both nodes as SANs change (LE sync in the job is best-effort; SPA Sync if it skips).
5. Catalog / SBC repoint (`move-tenant.sh` or Fleet Move job).
6. **Drain, then wipe source** — after verify, the job sits at `awaiting_cleanup`. Wait for phones to re-register on dest. You can leave the job page; reopen via Fleet → **Jobs** → **Open**, then **Wipe tenant on source** (full cascade + Commit). Do not start a second Move for the same wipe.

**Trunks do not move** — recreate or map on destination.

```bash
# source
sudo -u www-data php artisan tenant:export {tenant}
# dest
sudo -u www-data php artisan tenant:import /opt/pbx3/bkup/pbx3tenant.{shortuid}.*.zip
# Mac catalog
./move-tenant.sh --tenant-shortuid {s} --instance-id {DEST_KSUID} --fqdn {tenant.fqdn}
```

Lab worked example: **08jzwn** → **bzy54n**, bucket `08jzwn-pbx3`.

Rollback while source still has the tenant: reverse DNS + catalog before source delete.
