# Control host (gatekeeper on EC2)

Lab stood up **2026-07-14** — reference EC2 shape, not a forever AWS lock-in.

| Item | Lab value |
|------|-----------|
| Host | `control.pbx3.com` |
| SSH | `ubuntu@control.pbx3.com` |
| Health | `GET https://control.pbx3.com/health` |
| Auth | Fleet login `POST /api/v1/auth/login` (+ break-glass token ops-only) |
| First user | `fleet@pbx3.com` |
| IAM profile | `pbx3-control-gatekeeper` on bucket `08jzwn-pbx3` |
| SBC API | `PBX3_SBC_ADMIN_API_URL=https://sbc.pbx3.com/api` |

## Dynamic IP caveat

Without an Elastic IP, stop/start changes the public address:

1. Update DNS A for `control.pbx3.com`.
2. Refresh security-group `/32` allowlists (SPA/dev Mac, probe sources) as needed.

TLS: nginx + Let's Encrypt on the control host (see directory `CONTROL_HOST.md` for ops detail).
