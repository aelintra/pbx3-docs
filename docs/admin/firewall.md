# Firewall

Route: `/firewall`

Homes use **UFW** (not Shorewall). **One** allow-list covers **IPv4 and IPv6** (UFW dual-stack — there is no separate IPv6 panel). The panel edits that list; **Save** writes it and **Apply** / restart runs the baseline apply script.

1. Review the allow rows (protocol, port, source, comment). Source may be `any`, IPv4/CIDR, or IPv6/CIDR.
2. **Save** (persists `/etc/pbx3/firewall.allows.json`).
3. **Apply** / restart so UFW picks up the change.

!!! warning "Narrow SSH and API (fleet)"
    **Fleet** install leaves **:22** (SSH) and **:44300** (API) with Source **`any`** so you are not locked out. Change those rows to your ops/VPN CIDR(s), then Save and Apply. On cloud, also tighten the security group for the same ports. The SPA warning banner shows only while those ports are still wide open.

    **Solo / singleton** install defaults **:22, :44300, SIP, and RTP** to the detected **LAN CIDR** (widen in the panel if you need remote access).

## What the baseline opens

| Profile | Typical allows |
|---------|----------------|
| **Fleet** | SIP **5060** only from the **SBC**; API **44300** and SSH from **`any` at install** (narrow ASAP); RTP from **`any`**; ephemeral ACME **:80** during Let’s Encrypt |
| **Solo** | **22, 44300, 5060/5061, RTP** from the **LAN CIDR** at install (widen in panel if needed) |

Cloud security groups are **in addition** to UFW — open **44300**, **80** (ACME), and SIP as needed on both.

## After a SARK migrate

Customer backups do **not** carry host Shorewall rules. Post-load firewall is the same **UFW baseline** as greenfield. `fqdninspect` / `sipflood` are forced **off** by the ETL (they had no UFW equivalent). Re-enter any custom allows in this panel if the old box had extra ACCEPTs.
