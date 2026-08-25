# Firewall

Route: `/firewall`

Homes use **UFW** (not Shorewall). The panel edits a declarative allow-list; **Save** writes it and **Apply** / restart runs the baseline apply script.

1. Review the allow rows (protocol, port, source, comment).
2. **Save** (persists `/etc/pbx3/firewall.allows.json`).
3. **Apply** / restart so UFW picks up the change.

## What the baseline opens

| Profile | Typical allows |
|---------|----------------|
| **Fleet** | SIP **5060** only from the **SBC**; API **44300**; SSH; ephemeral ACME **:80** during Let’s Encrypt |
| **Solo** | Same idea plus LAN / operator-chosen sources for SIP |

Cloud security groups are **in addition** to UFW — open **44300**, **80** (ACME), and SIP as needed on both.

## After a SARK migrate

Customer backups do **not** carry host Shorewall rules. Post-load firewall is the same **UFW baseline** as greenfield. `fqdninspect` / `sipflood` are forced **off** by the ETL (they had no UFW equivalent). Re-enter any custom allows in this panel if the old box had extra ACCEPTs.
