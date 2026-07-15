# Sign in to the admin UI

PBX3 uses **one SPA** with two auth planes:

| Mode | Token | Used for |
|------|-------|----------|
| **Instance admin** | Sanctum (per node) | Tenants, extensions, Commit, Backup, Certificates |
| **Fleet console** | Gatekeeper session | Instances, Jobs, DIDs, Reconcile, Users |

Keep those separate (do not mix tokens).

## Instance admin (day-to-day)

### Solo

1. Open SPA → login form.
2. Email + password + API URL (`https://{node}:44300/api`).

### Fleet catalog picker

1. SPA built/configured with catalog URL (`VITE_INSTANCE_DIRECTORY_URL`).
2. **Refresh catalog** → pick an instance.
3. Sign in with **that node's** admin credentials.
4. Top bar shows connected instance label + FQDN.

### Lab endpoints

| Role | URL |
|------|-----|
| Golden API | `https://08jzwn.pbx3.com:44300/api` |
| Second node | `https://bzy54n.pbx3.com:44300/api` |
| Catalog JSON | `https://08jzwn-pbx3.s3.us-east-1.amazonaws.com/catalog/instance-index.json` |
| Gatekeeper | `https://control.pbx3.com` |
| SBC admin | `https://sbc.pbx3.com/admin` |

## Fleet console

1. From SPA Home / login chooser: **Fleet console**, or **Enter Fleet** (dual-hat) after instance login.
2. Sign in to Gatekeeper (email/password). Lab user: `fleet@pbx3.com` (password in ops secret store).
3. Nav stays locked until Sign in succeeds.
4. **Exit** returns to instance mode; **Logout** ends the fleet session.

Break-glass Bearer paste is ops-only under Advanced — prefer normal fleet login.

## Checks when login fails

See [Cannot log in](../troubleshooting/cannot-log-in.md).
