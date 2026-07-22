# Number dialects (PSTN formats)

Operators map each carrier Peer to a **number dialect** so dialled numbers and CLI survive Magrathea ↔ Gamma (and similar) format quirks. Inventory stays E.164 **digits**; the fleet wire between node and SBC is **+E.164**.

**Requirements:** `pbx3/pbx3-directory/docs/NUMBER_DIALECT_REQUIREMENTS.md` (same tree as other fleet specs).

## Why +E.164

ITU-T E.164 is the digit structure; for SIP routing across carriers and Teams-style peers, the userpart should include the leading **`+`** (e.g. `+441924918076`, US `+15556667777`). See [Twilio E.164](https://www.twilio.com/docs/glossary/what-e164).

## Where to set it

In **SBC Admin → Peering → Peers**:

1. Open the Magrathea / Gamma inbound or outbound Peer (not Asterisk destinations).
2. Set **Number dialect**:
   - **UK — Magrathea** — accepts national / +E.164 / IDD / digits; outbound +E.164; PAID + RPID
   - **UK — Gamma** — same inbound accept; outbound +E.164; PAID
   - **Strict +E.164** — Teams-style; inbound must already be `+…`
   - **None** — best-effort UK inbound; outbound leaves drouting strip/prefix alone
3. Save (triggers `dr_reload`).

Stored as `dialect=uk-magrathea` (etc.) in Peer `attrs` alongside `carrier=` / `role=`.

## Call path (summary)

| Leg | Behaviour |
|-----|-----------|
| Carrier → SBC | Normalize R-URI (+ From) per inbound Peer dialect → digit match on `dr_rules` / alias → **+E.164** toward Asterisk |
| Node → SBC | Egress should send **+E.164** dialled (fleet seed sets trunk `transform` `0:+44 00:+`) |
| SBC → carrier | After `do_routing(0)`, render dialled + CLI for **outbound** Peer dialect |

## Node / inroutes

After dialect normalize, inbound DIDs arrive on the node as **+E.164**. Prefer `inroutes.pkey` patterns that match `+44…` (or `44…`). National-only regexes may miss once the SBC rewrite is live.

Store extension / cluster / trunk CLIDs as **+E.164** when possible so outbound CLIP does not need a second national→international pass.

## Lab checklist

| # | Case | How |
|---|------|-----|
| M1 | Magrathea inbound national / + / IDD | Set inbound Peer `uk-magrathea`; place test calls; confirm Asterisk R-URI `+44…` |
| M2 | Magrathea outbound | Outbound Peer `uk-magrathea`; dial PSTN; confirm R-URI/CLI `+` and PAID |
| G1 | Gamma outbound | Outbound Peer `uk-gamma` |
| X1 | Cross-carrier | Magrathea DID inbound + Gamma egress; dialled/CLI rendered for Gamma |

Offline matrices: `pbx3sbc-admin` PHPUnit `NumberDialectTest`.

## Carrier docs

- Magrathea [LI Agreement](https://www.magrathea-telecom.co.uk/wp-content/uploads/2018/09/LI-Agreement-1.pdf), [network/presentation guidance](https://www.magrathea-telecom.co.uk/wp-content/uploads/2018/11/Guidance-on-Network-and-Presentation-numbers.pdf)
- Gamma SIP trunk CPE notes (R-URI/To national or +E.164; From/PAID national or +E.164)
