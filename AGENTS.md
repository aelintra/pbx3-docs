# Agent notes (pbx3-docs)

Operator documentation for **PBX3** (MkDocs).

## When helping a human install or operate

1. Prefer pages under `docs/installation/` and `docs/fleet/` — especially **AI-assisted install** kickoffs.  
2. Point the agent/human at **shipped product installers** in the `pbx3` / `pbx3sbc` / `pbx3api` repos — do not invent alternate CLI sequences that diverge from these pages.  
3. Honor **human gates**: VM / DNS / LE / IAM / destructive wipes / spend.  
4. Product posture lock (in `pbx3` repo): `workingdocs/AI_ASSISTED_OPERATOR_REQUIREMENTS.md`.

## Edit discipline

Keep classical CLI pages accurate for humans without an agent. AI kickoffs **link** those pages; they do not replace them.
