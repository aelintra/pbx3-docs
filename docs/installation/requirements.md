# Requirements

## Platform

- **Ubuntu 24.04 LTS**
- Root / sudo
- Appliance mindset: packages land under `/opt/pbx3` and `/opt/pbx3api`

## Network (typical)

| Direction | Ports / use |
|-----------|-------------|
| Inbound | **22** SSH · **44300** API · **80** Let's Encrypt HTTP-01 (during issue/renew) · SIP per design |
| Outbound | **443** (apt, S3, Let's Encrypt) |

## Build machine (optional)

If you build Debian packages yourself:

```bash
sudo apt install dpkg-dev debhelper build-essential
```

## Lab reference

| Item | Lab-golden |
|------|------------|
| Example FQDN | `08jzwn.pbx3.com` |
| Org bucket (fleet) | `08jzwn-pbx3` |
| Region | `us-east-1` |

Solo trial does **not** require S3. Fleet does — see [Cloud / S3](../cloud/bucket-layout-cors.md).
