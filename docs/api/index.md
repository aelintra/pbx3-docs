# Instance API

## Background

**pbx3api** is a JSON HTTPS API on each PBX3 instance. It lets you do programmatically what the admin SPA does on that node.

Typical uses:

- Automate extensions, routes, tenants, and other panel resources
- Drive a remote Asterisk/pbx3 from a management layer or third-party integration
- Build custom UIs on top of the same Sanctum-authenticated surface the SPA uses

This section documents the **instance** API (`:44300`). The **Gatekeeper** fleet control plane (`control.pbx3.com`) is a separate API and is not covered here except where instance fleet-token routes are noted.

## Requirements

- A PBX3 node with **pbx3** + **pbx3api** installed
- PHP **8.2+**, 64-bit (x86 or ARM)
- Laravel 11 framework (API application under `/opt/pbx3api`)

## Base URL

```
https://{node-fqdn-or-ip}:44300/api
```

Lab golden example: `https://08jzwn.pbx3.com:44300/api`

All paths in this guide are relative to that base (e.g. `POST /auth/login` → `https://…:44300/api/auth/login`).

## Conventions (read first)

### Authentication planes

| Plane | Header | Used for |
|-------|--------|----------|
| **Sanctum** | `Authorization: Bearer <accessToken>` | Almost all instance routes |
| **Fleet service token** | `Authorization: Bearer <PBX3_FLEET_SERVICE_TOKEN>` | `/api/fleet/*` only (mobility / fleet create-delete) |

Unauthenticated callers may only use **`POST /auth/login`**. Everything else needs a Bearer token of the appropriate plane.

### Abilities (Sanctum)

Each user/token has an **array** of ability strings. Lexicon:

| Ability | Meaning |
|---------|---------|
| `admin` | Full instance access (users, trunks/routes, system, all tenants, AMI, …) |
| `tenant` | Tenant ops within `allowed_clusters` (extensions, queues, CoS, CDR, commit, …) |
| `recordings` | Listen/download recordings within `allowed_clusters` (additive) |

Route groups use either `abilities:admin` (must have admin) or `ability:admin,tenant` / `ability:admin,recordings` (any of the listed abilities).

### Tenant identity (shortuid / KSUID)

For tenant-scoped rows (extensions, queues, routes, …):

- DB **`cluster`** stores the tenant **shortuid** (not the human tenant name/`pkey`)
- Clients often send the tenant **pkey**; the API resolves it to shortuid for storage
- Row identity is **`id`** (KSUID). Prefer **`shortuid`** in URLs; binding also accepts `id` / `pkey` fallbacks where implemented
- Do **not** send `id` or `shortuid` on create — the API generates them
- Responses may include display field **`tenant_pkey`**

### Schemas (SPA mutability)

**`GET /schemas`** returns per-resource `read_only`, `updateable`, and `defaults` derived from the live DB plus controller updateable lists. Use it (or the controller) to verify POST/PUT bodies when this digest and code diverge. It is **not** an OpenAPI document.

### Fleet tenant lifecycle

On fleet-joined nodes, creating/deleting tenants via Sanctum `POST/DELETE /tenants` may be locked. Prefer **`/api/fleet/tenants`** with the fleet service token. See [Endpoint reference — Fleet](reference.md#fleet-service-token).

## Read next

1. [Methods and notation](overview.md)
2. [Authentication](auth.md)
3. [Endpoint reference](reference.md)
