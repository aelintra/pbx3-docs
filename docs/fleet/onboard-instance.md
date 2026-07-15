# Onboard a second instance

Join another healthy node to the org bucket and catalog.

## Before you start

- Node installed; `GET /up` → 200
- Stable `globals.id` / shortuid / fqdn
- No fleet `.env` yet on the node
- Ops Mac AWS identity is **not** a node role (`pbx3-node-*`)

## Preferred script (Mac)

```bash
cd pbx3/pbx3-directory/tools
export PBX3_ORG_BUCKET=08jzwn-pbx3
./onboard-fleet-instance.sh \
  --instance-id i-XXXXXXXX \
  --ssh ubuntu@NODE.pbx3.com \
  --ssh-key ~/path/to/key.pem \
  --region us-east-1
# dry-run first:
# ./onboard-fleet-instance.sh ... --dry-run
```

## What it covers

IAM policy/role/profile → attach EC2 → SSH `.env` (`PBX3_ORG_BUCKET`, backup upload) → S3 smoke → register catalog → SPA shows a second row.

## Manual debug phases

Same order as the script; stop after any red preflight. On node:

```bash
sudo php artisan pbx3:fleet-preflight
```

## Remove from catalog

```bash
./unregister-instance.sh --id {KSUID}          # soft
./unregister-instance.sh --id {KSUID} --remove # hard catalog row
```

Does not delete S3 backups or stop calls.

Also: [Cloud / S3](../cloud/bucket-layout-cors.md), [Agent-assisted](agent-assisted.md).
