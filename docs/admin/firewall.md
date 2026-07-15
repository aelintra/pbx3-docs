# Firewall

Route: `/firewall`

1. Edit IPv4 / IPv6 rules.
2. **Save**.
3. **Restart** firewall (applies rules).

When FQDN inspect is enabled, restart refreshes `pbx3_inline_fqdn` from the tenant list.

Cloud security groups are **in addition** to Shorewall — open **44300**, **80** (ACME), SIP as needed on both.
