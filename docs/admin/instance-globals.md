# Instance globals

Route: `/sysglobals`

## Safe vs careful

| Usually OK | Treat as install / migration |
|------------|------------------------------|
| Sitename / display fields | `domain`, instance FQDN |
| Feature toggles you understand | `globals.id` / shortuid |

`globals.id` (KSUID) is the S3 prefix and catalog identity — do not casually change it.

## FQDN inspect

Legacy Shorewall STRING matching on SIP. **Fleet homes on UFW:** leave **off** — SIP safety is **SBC + “5060 from SBC only”**. Solo/direct may still show the toggle; it does not drive UFW rules. Prefer strong credentials and source allows in [Firewall](firewall.md).
