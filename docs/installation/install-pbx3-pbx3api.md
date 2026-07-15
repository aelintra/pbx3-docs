# Install pbx3 and pbx3api

Greenfield order on a clean Ubuntu 24.04 node.

## 1. Install pbx3 package

```bash
sudo apt install ./pbx3_*.deb
```

## 2. First-run installer (pbx3)

```bash
sudo /opt/pbx3/scripts/installer.sh
# or constrain identity:
# sudo DOMAIN_TLD=example.com /opt/pbx3/scripts/installer.sh
# sudo INSTANCE_FQDN=node.example.com /opt/pbx3/scripts/installer.sh
```

Creates SQLite DB, hostname/shorewall baseline, and `globals` identity (`id` KSUID, `shortuid`, `domain`, `fqdn`).

Default apex when unset: `pbx3.com`.

## 3. Deploy and install pbx3api

Put the API tree at `/opt/pbx3api`, then:

```bash
sudo /opt/pbx3api/scripts/installer.sh
```

API listens on **`https://<host>:44300`** (snakeoil until Let's Encrypt).

## 4. DNS

Point an A record for `globals.fqdn` at the node's public IP.

## 5. Certificates

Issue the first LE cert — see [First Let's Encrypt certificate](../tls/first-letsencrypt.md).

## Verify

```bash
curl -sS -o /dev/null -w '%{http_code}\n' -k https://127.0.0.1:44300/up
# expect 200
```

Then open the SPA and [sign in](../getting-started/sign-in.md).

## Next

- Solo: [Admin guide](../admin/home-dashboard.md)
- Fleet: [Onboard](../fleet/onboard-instance.md) after IAM/DNS ready
