# Number dialects (PSTN formats)

Operators map each carrier Peer to a **number dialect** so dialled numbers and CLI survive ITSP format quirks (for example **Magrathea Telecom** ↔ **Gamma**). Inventory stays E.164 **digits**; the fleet wire between node and SBC is **+E.164** (`+` + country code of the **node’s serving country**, not hard-coded `+44`).

**Who does what (policy):** `pbx3/pbx3-directory/docs/NUMBER_WIRE_POLICY.md`  
**Requirements:** `pbx3/pbx3-directory/docs/NUMBER_DIALECT_REQUIREMENTS.md` (same tree as other fleet specs).

**Naming:** **The SBC** = our edge product (admin UI / OpenSIPS). **Magrathea Telecom** and **Gamma** = example UK **ITSP** Peers on that edge — not names for the SBC itself.

## Why +E.164

ITU-T E.164 is the digit structure; for SIP routing across carriers and Teams-style peers, the userpart should include the leading **`+`** (e.g. UK `+441924918076`, US `+15556667777`). See [Twilio E.164](https://www.twilio.com/docs/glossary/what-e164).

## Two layers (do not conflate)

Full policy: **`NUMBER_WIRE_POLICY.md`**. Short form:

| Layer | Problem | Where (Phase 1 now) |
|-------|---------|---------------------|
| **Subscriber dial habit** | National / IDD access codes from the phone (UK `0…` / `00…`, US `1…` / `011…`) | **Node** — Egress trunk transformation mask (DNID only) |
| **Carrier wire format** | What Magrathea Telecom / Gamma / etc. accept on R-URI and PAID/CLI | **SBC** — Peer Number dialect (+ optional strip / pri_prefix) |

Phase 2 may move habit normalize to the SBC when gated; carrier face stays on the SBC always.

### Node — DNID vs CLID

- **DNID (dialled number):** Egress / trunk **transformation mask** rewrites toward `+CC…` before the INVITE hits the SBC (pbx3cagi `Mangle`).
- **CLID:** Chosen on the node (`outboundClip` — extension / cluster / trunk) and sent **as stored** — masks do **not** rewrite CLI today.
- Store CLIDs as **`+CC…`** for the country that node serves so “as-is” already matches fleet wire. Carrier PAID shape (e.g. Gamma insists E.164 or +E.164) is enforced on the **SBC** outbound Peer dialect.

### Node — access codes by serving country

| Serving country | National habit | Overseas IDD | Typical Egress `transform` |
|-----------------|----------------|--------------|----------------------------|
| UK | `0` + NSN | `00` + country + NSN (e.g. `0015139266349`) | `00:+ 0:+44` (fleet seed; longer prefix first) |
| US / NANP | `1` + 10 digits (often leave as-is) | `011` + country + NSN (e.g. `011441924918076`) | e.g. `011:+` when US fleet seed exists — **do not** apply UK `0:+44` |

Examples after node transform: `0015139266349` → `+15139266349`; `011441924918076` → `+441924918076`.

## Where to set carrier dialect (SBC)

In **SBC Admin → Peering → Peers**:

1. Open the Magrathea Telecom / Gamma inbound or outbound Peer (not Asterisk destinations).
2. Set **Number dialect** (a **format recipe**, not a carrier brand — see requirements §5.3):
   - **UK — Magrathea** — UK multi-accept; outbound +E.164; PAID + RPID (recipe id `uk-magrathea`; named for the Magrathea Telecom CLI matrix, not the SBC)
   - **UK — Gamma** — same inbound accept; outbound +E.164; PAID (From aligned)
   - **Strict +E.164** — Teams-style; inbound must already be `+…`
   - **None** — best-effort UK inbound; outbound leaves drouting strip/prefix alone
3. Optional **Advanced → strip / pri_prefix** — stock OpenSIPS drouting digit massage on outbound R-URI only (count to strip, then digit string to prepend). Prefer a dialect when CLI/PAID rules matter.
4. Save (triggers `dr_reload`).

Stored as `dialect=uk-magrathea` (etc.) in Peer `attrs` alongside `carrier=` / `role=`. Do **not** expect a new dialect id for every ITSP or country — reuse a recipe and set country via serving-country / `default_cc` when that lands on the Peer.

**Target (requirements §5.4):** ops must be able to **compose a new profile** from shipped parsers/renderers (and `default_cc`) without an OpenSIPS tip. Today only the built-in recipe enum is live; slot/profile editing is the next dialect slice.

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
| M1 | Magrathea Telecom inbound national / + / IDD | Set inbound Peer `uk-magrathea`; place test calls; confirm Asterisk R-URI `+44…` |
| M2 | Magrathea Telecom outbound | Outbound Peer `uk-magrathea`; dial PSTN; confirm R-URI/CLI `+` and PAID |
| G1 | Gamma outbound | Outbound Peer `uk-gamma`; confirm PAID is +E.164 (or E.164 digits per contract) |
| X1 | Cross-carrier | Magrathea Telecom DID inbound + Gamma egress; dialled/CLI rendered for Gamma |
| N1 | UK phone → US | Dial `001…`; node transform → `+1…` before SBC |

Offline matrices: `pbx3sbc-admin` PHPUnit `NumberDialectTest`.

## Carrier docs

- Magrathea Telecom [LI Agreement](https://www.magrathea-telecom.co.uk/wp-content/uploads/2018/09/LI-Agreement-1.pdf), [network/presentation guidance](https://www.magrathea-telecom.co.uk/wp-content/uploads/2018/11/Guidance-on-Network-and-Presentation-numbers.pdf)
- Gamma SIP trunk CPE notes (R-URI/To national or +E.164; From/PAID national or +E.164)
