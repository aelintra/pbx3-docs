# Agent-assisted onboard / rebuild

**Interim** until orchestrated S10.7 / S8.9 ships. Human owns AWS/DNS approvals; an AI coding agent drives Mac scripts + SSH.

Broader posture (install + Lab): [AI-assisted operations](../getting-started/ai-assisted.md) · [AI-assisted install](../installation/ai-assisted-install.md).  
Lock: `pbx3/workingdocs/AI_ASSISTED_OPERATOR_REQUIREMENTS.md`.

## Kickoff prompt (paste to agent)

```text
Rebuild fleet node from S3 — follow REBUILD_INSTANCE_RUNBOOK.md on main (pbx3-directory/docs).
Instance KSUID: {ksuid}. Org bucket: {bucket}. Region: {region}.
Use latest S3 backup unless I specify a stamp.
Ask before: terminating EC2, DNS cutover, IAM-impacting changes on production.
After restore: pbx3:fleet-preflight must be all green before we call it done.
Never put ops AWS credentials on the node or in the SPA.
```

## Human gates

- Launch / terminate EC2
- Hold ops AWS credentials (never bake onto nodes)
- DNS / LE cutover
- Final Commit and accept preflight green

## Tools home

`pbx3/pbx3-directory/tools/` — `onboard-fleet-instance.sh`, `fetch-latest-instance-backup.sh`, register/unregister helpers.

Mac setup: `OPERATOR_MAC_SETUP.md` in directory docs (SSH + AWS CLI).
