# AI-assisted operations

PBX3 is built so an **AI coding agent** (Cursor, Copilot Chat agent mode, Claude Code, etc.) can do much of the operator work **with you** — install, Lab bring-up, fleet onboard — instead of you copying long CLI runbooks by hand.

Canonical install pages and scripts remain the **source of truth**. The agent should **run those paths**, not invent a parallel installer.

!!! tip "Requirements lock"
    Product posture: `pbx3` repo → `workingdocs/AI_ASSISTED_OPERATOR_REQUIREMENTS.md`.

## How it works

1. Open this docs site (or the cloned repo) in your agent-capable editor.  
2. Paste a **kickoff prompt** from [AI-assisted install](../installation/ai-assisted-install.md) or [Agent-assisted onboard / rebuild](../fleet/agent-assisted.md).  
3. Fill `{placeholders}` (host, email, domain, …).  
4. Approve **human gates** when the agent stops (cloud VM, DNS, certificates, spend, destructive wipes).  
5. Confirm the **done when** check (e.g. `/up` returns 200).

## What you still own

| You | Agent |
|-----|--------|
| Cloud / DNS / certificate approvals | Driving documented scripts over SSH |
| Passwords and ops IAM (never in the browser) | Reading MkDocs + in-repo runbooks |
| “Is this production?” judgment | Wiring worksheet env vars and re-running failed steps |

## Prefer classical docs?

Every AI kickoff points at the same pages a human would follow alone — start at [Install requirements](../installation/requirements.md) or [Solo trial](solo-trial.md).
