# Fleet overview

Many interchangeable **instances**. A **tenant** lives on one instance at a time and can move without reconfiguring phones.

| Piece | Job |
|-------|-----|
| **SBC** | Single SIP front door; routes by tenant domain |
| **Instances** | Asterisk + API + tenant data |
| **Org S3** | Catalog, backups, recordings — not on the call path |
| **Gatekeeper** | Fleet control plane (`control.pbx3.com` in lab) |
| **SPA Fleet mode** | Operator UI for catalog / jobs / DIDs |

Calls keep working if Gatekeeper or S3 is down.

## Solo vs fleet

| Solo | Fleet |
|------|-------|
| One node; S3 optional | Shared bucket + catalog + usually SBC |
| Instance Sanctum only | + Gatekeeper fleet token |

See also the schematic on [What is PBX3?](../getting-started/what-is-pbx3.md).

## Lab map

Phones → **sbc.pbx3.com** → **08jzwn** / **bzy54n**; bucket **`08jzwn-pbx3`**; Gatekeeper **control.pbx3.com**.
