# First Let's Encrypt certificate

## Prerequisites

- A (and optional AAAA) for **every** name that will appear on the cert → this node's public IP
- Port **80** reachable from the internet during the challenge (cloud SG **and** Shorewall)

## CLI

```bash
sudo /opt/pbx3/scripts/le-instance-bootstrap.sh your@email.com
# staging: PBX3_LE_STAGING=1 sudo /opt/pbx3/scripts/le-instance-bootstrap.sh your@email.com
```

## SPA

**Certificates** → Get certificate (email + setup). Top line should show **Let's Encrypt** in use.

## Lab ACME reachability check

```bash
curl -v --connect-timeout 5 http://08jzwn.pbx3.com/.well-known/acme-challenge/test
# TCP connect OK; 404 on a fake path is fine
```

## Trusted API check

```bash
curl -sS -o /dev/null -w '%{http_code}\n' https://08jzwn.pbx3.com:44300/up
# 200 without -k
```

If it fails, see [Certificate / HTTPS errors](../troubleshooting/certificate-errors.md).
