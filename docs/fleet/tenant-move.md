# Tenant move

High-level order (panel Jobs / Move wizard may wrap this):

1. Prep destination capacity / trunk mapping.
2. Export on source → import on destination → **Commit** → test on dest **before** DNS.
3. DNS A for tenant FQDN → destination.
4. Cert **Sync** on both nodes as SANs change.
5. Catalog / SBC repoint (`move-tenant.sh` or Fleet Move job).
6. Delete tenant on source → Sync / Commit.

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
