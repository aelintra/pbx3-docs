# Node IAM (writer role)

## Actors

| Actor | Scope |
|-------|--------|
| `pbx3-node-{shortuid}` | `instances/{own_ksuid}/*` only |
| Mac ops / Gatekeeper | `catalog/*`, `instances/*`, `tenants/*` as designed |
| **Never** | Node role writing `catalog/*`; long-lived ops keys on the node |

No blanket `tenants/*` on node writer policies.

## Node `.env` (fleet)

```env
AWS_DEFAULT_REGION=us-east-1
PBX3_ORG_BUCKET=08jzwn-pbx3
PBX3_DIRECTORY_BACKUP_UPLOAD=true
```

**Delete** empty `AWS_ACCESS_KEY_ID=` / `AWS_SECRET_ACCESS_KEY=` lines so the instance profile is used.

## Preflight

```bash
sudo php artisan pbx3:fleet-preflight
```

Includes deny-probe expectations for over-broad `tenants/*` access.

Apply live policies with directory tools such as `apply-node-s3-writer-policy.sh` from a Mac ops identity.
