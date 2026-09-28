# AI-assisted operations

If you already use an **AI coding agent** (Cursor, Copilot agent mode, Claude Code, etc.), PBX3 documents a **co-pilot** path: the agent drives **published** installers and runbooks **with you**. It is optional. The classical MkDocs/CLI pages remain a **complete** install path with no agent required.

PBX3 itself was **designed by humans** in mid/late 2025; most of the implementation was written in Cursor under **human-directed AI** assistance. That is how the product was built — and the same limits apply when you use an agent to operate it.

!!! warning "Not unattended — and not a panacea"
    This is not “AI installs PBX3 for you.” You approve destructive and spend-adjacent steps (VMs, DNS, certificates, IAM, wipes, PSTN spend). Proceed **one phase at a time** and verify checks; that limits agent errors but does **not** eliminate them — current coding agents still fail. Third-party chat logs can retain whatever you paste — put secrets in your **shell environment / SSH agent**, not in the prompt.

    **Canonical path:** classical [Install](../installation/requirements.md) pages. Use this co-pilot path only if you already work with an agent IDE.

!!! tip "Requirements lock"
    Product posture: `pbx3` repo → `workingdocs/AI_ASSISTED_OPERATOR_REQUIREMENTS.md` (including cons mitigations).

## How it works

1. Open this docs site (or the cloned repo) in your agent-capable editor.  
2. Paste a **kickoff prompt** from [AI-assisted install](../installation/ai-assisted-install.md) or [Agent-assisted onboard / rebuild](../fleet/agent-assisted.md).  
3. Fill non-secret `{placeholders}`; set passwords/keys in your shell and tell the agent the **variable names** only.  
4. Approve **human gates** when the agent stops.  
5. Confirm the **done when** check (e.g. `/up` returns 200).  
6. Prefer **one step or short phase at a time** — verify the check, then continue. Do not ask the agent to “finish the whole install” in one go; that is how silent drift accumulates.  
7. If something fails: bisect on the **canonical page** the kickoff cites — that is what maintainers treat as ground truth.

Residual mistakes after careful stepwise use are expected with today’s agents. Treat that as an operating hazard, not a PBX3 defect, unless the documented script/page is wrong.

## What you still own

| You | Agent |
|-----|--------|
| Cloud / DNS / certificate approvals | Driving documented scripts over SSH |
| Passwords and ops IAM (never in the browser or chat) | Reading MkDocs + in-repo runbooks |
| “Is this production?” judgment | Wiring worksheet env vars and re-running failed steps |
| Signaling / Peer / dialplan changes on live systems | Stopping to ask (see gates) |

## Prefer classical docs?

Use [Install requirements](../installation/requirements.md) or [Solo trial](solo-trial.md) alone — equal citizenship, not a fallback ghetto.
