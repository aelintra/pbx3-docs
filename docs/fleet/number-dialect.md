# Number dialects (PSTN formats)

Operators map each carrier Peer to a **number dialect** so dialled numbers and CLI survive Magrathea ↔ Gamma (and similar) format quirks. Inventory stays E.164 **digits**; the fleet wire between node and SBC is **+E.164** (`+` + country code of the **node’s serving country**, not hard-coded `+44`).

**Requirements:** `pbx3/pbx3-directory/docs/NUMBER_DIALECT_REQUIREMENTS.md` (same tree as other fleet specs).

## Why +E.164

ITU-T E.164 is the digit structure; for SIP routing across carriers and Teams-style peers, the userpart should include the leading **`+`** (e.g. UK `+441924918076`, US `+15556667777`). See [Twilio E.164](https://www.twilio.com/docs/glossary/what-e164).

## Two layers (do not conflate)

| Layer | Problem | Where |
|-------|---------|--------|
| **Subscriber dial habit** | National / IDD access codes from the phone (UK `0…` / `00…`, US `1…` / `011…`) | **Node** — Egress trunk transformation mask (DNID only) |
| **Carrier wire format** | What Magrathea/Gamma/etc. accept on R-URI and PAID/CLI | **SBC** — Peer Number dialect (+ optional strip / pri_prefix) |

### Node — DNID vs CLID

- **DNID (dialled number):** Egress / trunk **transformation mask** rewrites toward `+CC…` before the INVITE hits the SBC (pbx3cagi `Mangle`).
- **CLID:** Chosen on the node (`outboundClip` — extension / cluster / trunk) and sent **as stored** — masks do **not** rewrite CLI today.
- Store CLIDs as **`+CC…`** for the country that node serves so “as-is” already matches fleet wire. Carrier PAID shape (e.g. Gamma insists E.164 or +E.164) is enforced on the **SBC** outbound Peer dialect.

### Node — access codes by serving country

| Serving country | National habit | Overseas IDD | Typical Egress `transform` |
|-----------------|----------------|--------------|----------------------------|
| UK | `0` + NSN | `00` + country + NSN (e.g. `0015139266349`) | `0:+44 00:+` (current fleet seed) |
| US / NANP | `1` + 10 digits (often leave as-is) | `011` + country + NSN (e.g. `011441924918076`) | e.g. `011:+` when US fleet seed exists — **do not** apply UK `0:+44` |

Examples after node transform: `0015139266349` → `+15139266349`; `011441924918076` → `+441924918076`.

## Where to set carrier dialect (SBC)

In **SBC Admin → Peering → Peers**:

1. Open the Magrathea / Gamma inbound or outbound Peer (not Asterisk destinations).
2. Set **Number dialect**:
   - **UK — Magrathea** — accepts national / +E.164 / IDD / digits; outbound +E.164; PAID + RPID
   - **UK — Gamma** — same inbound accept; outbound +E.164; PAID
   - **Strict +E.164** — Teams-style; inbound must already be `+…`
   - **None** — best-effort UK inbound; outbound leaves drouting strip/prefix alone
3. Optional **Advanced → strip / pri_prefix** — stock OpenSIPS drouting digit massage on outbound R-URI only (count to strip, then digit string to prepend). Prefer a dialect when CLI/PAID rules matter.
4. Save (triggers `dr_reload`).

Stored as `dialect=uk-magrathea` (etc.) in Peer `attrs` alongside `carrier=` / `role=`.

## Call path (summary)

| Leg | Behaviour |
|-----|-----------|
| Phone → node | Tenant dial habit (national / IDD) |
| Node → SBC | DNID via transform → **`+CC…`**; CLID as stored (prefer `+CC…`) |
| Carrier → SBC | Normalize per inbound Peer dialect → digit match → **+E.164** toward Asterisk |
| SBC → carrier | After `do_routing(0)`, render dialled + CLI/PAID for **outbound** Peer dialect |

## Node / inroutes

After dialect normalize, inbound DIDs arrive on the node as **+E.164**. Prefer `inroutes.pkey` patterns that match `+CC…` (e.g. `+44…`) or digit E.164. National-only regexes may miss once the SBC rewrite is live.

## Lab checklist

| # | Case | How |
|---|------|-----|
| M1 | Magrathea inbound national / + / IDD | Set inbound Peer `uk-magrathea`; place test calls; confirm Asterisk R-URI `+44…` |
| M2 | Magrathea outbound | Outbound Peer `uk-magrathea`; dial PSTN; confirm R-URI/CLI `+` and PAID |
| G1 | Gamma outbound | Outbound Peer `uk-gamma`; confirm PAID is +E.164 (or E.164 digits per contract) |
| X1 | Cross-carrier | Magrathea DID inbound + Gamma egress; dialled/CLI rendered for Gamma |
| N1 | UK phone → US | Dial `001…`; node transform → `+1…` before SBC |

Offline matrices: `pbx3sbc-admin` PHPUnit `NumberDialectTest`.

## Carrier docs

- Magrathea [LI Agreement](https://www.magrathea-telecom.co.uk/wp-content/uploads/2018/09/LI-Agreement-1.pdf), [network/presentation guidance](https://www.magrathea-telecom.co.uk/wp-content/uploads/2018/11/Guidance-on-Network-and-Presentation-numbers.pdf)
- Gamma SIP trunk CPE notes (R-URI/To national or +E.164; From/PAID national or +E.164)
