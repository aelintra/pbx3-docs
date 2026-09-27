# Feature codes

Star codes you dial from a registered phone (or softphone) on a **tenant**. They run in that tenant’s dialplan — not across Site Group prefixes (no `81*50*…` style).

Most codes end with a trailing `*`. Digits in braces are what you dial after the code.

Format column uses:

| Token | Meaning |
|-------|---------|
| `{ext}` | Extension number (tenant `ext_len`, usually 3–4 digits) |
| `{nn}` | One or more digits (see notes) |
| `{greet}` | Four-digit greeting number |
| *(prompt)* | Code alone; phone prompts for the rest |

Passwords (where noted) come from the **tenant** settings (system / spy pass), not from the phone’s voicemail PIN unless stated.

---

## Do not disturb and call forward

| Dial | What it does |
|------|----------------|
| `*18*` | DND **on** — direct calls to this extension go to voicemail |
| `*19*` | DND **off** |
| `*20*` | DND **toggle** |
| `*21*{ext}` | Call forward **immediate** (CFIM) to `{ext}` |
| `*21*` | Clear CFIM |
| `*22*{ext}` | Call forward **busy** (CFBS) to `{ext}` |
| `*22*` | Clear CFBS |
| `*23*` | Clear **all** call forwards (CFIM + CFBS) |
| `*26*` | Clear ring-delay override (no timeout → ring until answer / other fail path) |
| `*26*{nn}` | Set ring delay before fail-over (seconds; typically 1–2 digits) |

---

## Timers (open / closed)

Requires the tenant **system** password when prompted (`*30*` / `*31*`).

| Dial | What it does |
|------|----------------|
| `*30*` | Force schedule to **AUTO** (resume day timers) — instance master |
| `*31*` | Force schedule to **CLOSED** — instance master |
| `*33*` | Force this **tenant** to AUTO / open |
| `*34*` | Force this **tenant** to CLOSED |

BLF keys tied to `MASTER` or the tenant open/closed hint toggle the same state without dialling.

Day-timer calendar behaviour (modes, profiles): [Day timers and route profiles](day-timers-and-profiles.md).

---

## Voicemail

| Dial | What it does |
|------|----------------|
| `*50*` | Retrieve voicemail for **this** extension (asks for mailbox password) |
| `*51*` | Retrieve voicemail for **another** mailbox (asks for mailbox number + password) |
| `*{ext}` | Leave a message in `{ext}`’s mailbox (or transfer a live call straight to that box) |
| `vm{ext}` | Open voicemail main for `{ext}` (hint / BLF style) |

---

## Greetings

| Dial | What it does |
|------|----------------|
| `*60*{greet}` | Record system greeting `{greet}` (exactly **4** digits). Prompts for system password, then record / save. |

There is no supported `*61*` “listen” shortcode in the current dialplan.

---

## Agents and queues

Dial the code, then follow the prompts (agent ID, then agent password where required).

| Dial | What it does |
|------|----------------|
| `*63*` | **Pause** agent — no new queue calls while paused |
| `*64*` | **Resume** agent |
| `*65*` | Agent **login** — start taking queue calls on this phone |
| `*66*` | Agent **logout** |

Queues / agents panels: [Queues, IVRs, and agents](queues-ivrs-agents.md).

---

## Supervision (ChanSpy)

Prompts for the tenant **spy** password, then spies on the target’s live channel.

| Dial | What it does |
|------|----------------|
| `*67*{ext}` | ChanSpy **whisper** — hear the call; whisper to the agent only |
| `*68*{ext}` | ChanSpy — listen only (no whisper) |

Target is an extension on **this** tenant. Cross-tenant spy must fail closed.

---

## Call pickup, park, hold

| Dial / action | What it does |
|---------------|----------------|
| `*8{ext}` | **Directed pickup** — answer a ringing `{ext}` you are allowed to pick up (call/pickup groups) |
| `*8` (in-call feature) | Blind pickup of a ringing call in your pickup group (handset feature digit; idle dial of bare `*8` is not a dialplan destination) |
| Transfer to `*900` | **Park** the call (default lot) |
| `901`–`903` | Retrieve a parked call from the default lot positions |
| Hold / Xfer keys | Handset features — hold plays MOH; transfer behaviour depends on the phone |

Default lot size and park extension can be changed per tenant via parking overlay; stock template uses `*900` / `901`–`903`.

---

## Diagnostics

| Dial | What it does |
|------|----------------|
| `*52*` | Echo test (audio path check) |
| `*55*` | Speak time and date |
| `*56*` | Speak this extension’s number |

---

## NANP-style aliases

North American vertical-service-style numbers that map onto the codes above:

| Dial | Maps to |
|------|---------|
| `*60` | Time/date (`*55*`) |
| `*65` | Speak extension (`*56*`) |
| `*72` + destination | CFIM on (same family as `*21*`) |
| `*73` | CFIM off |
| `*77` + 4 digits | Record greeting (same family as `*60*`) |
| `*78` | DND on |
| `*79` | DND off |
| `*90` + destination | CFBS on |
| `*91` | CFBS off |
| `*97` | Local voicemail (`*50*`) |
| `*98` | Echo test (`*52*`) |

Prefer the `*NN*` forms above when documenting or training users.

---

## Not supported / avoid

| Code | Notes |
|------|--------|
| Follow-me (`*27*` family) | Handler stub — do not rely on it |
| Wake-up (`*24*`) | Legacy; not maintained for current PJSIP fleets |
| `*99…` → listen greeting | Not a supported listen path |

---

## See also

- [Extensions](extensions.md)
- [Queues, IVRs, and agents](queues-ivrs-agents.md)
- [Day timers and route profiles](day-timers-and-profiles.md)
