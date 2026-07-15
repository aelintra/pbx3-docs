# Instance globals

Route: `/sysglobals`

## Safe vs careful

| Usually OK | Treat as install / migration |
|------------|------------------------------|
| Sitename / display fields | `domain`, instance FQDN |
| Feature toggles you understand | `globals.id` / shortuid |

`globals.id` (KSUID) is the S3 prefix and catalog identity — do not casually change it.

## FQDN inspect

Enable when internet SIP uses URI string matching for tenant FQDNs (common with SBC + multi-tenant). Regenerates related firewall / Asterisk pieces on tenant changes and Commit.
