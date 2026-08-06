# Authentication

pbx3api uses **Laravel Sanctum** (Bearer tokens, similar to GitHub personal access tokens).

**Ways to get a token:**

- An administrator creates a user with email/password. You **login** and receive a Bearer token for the `Authorization` header.
- An administrator issues you a Bearer token directly (no login required for subsequent calls).

As an **unauthenticated** user, the only endpoint you can call is **`POST /auth/login`**. All other Sanctum-protected requests need `Authorization: Bearer <accessToken>`.

## Abilities

Each user has an **abilities** array. The token issued at login carries the same list. Routes are gated by ability middleware.

| Ability | Description |
|---------|-------------|
| `admin` | Full instance access: users, trunks/routes, system, all tenants |
| `tenant` | Tenant ops within `allowed_clusters` |
| `recordings` | Listen/download recordings within `allowed_clusters` |

Valid names are defined in the API ability lexicon (`config/abilities.php`). Only those names are accepted on register/update.

Tenant and recordings users are further scoped by **`allowed_clusters`** (cluster/tenant shortuids or identifiers the user may touch). Controllers enforce row-level cluster access.

Use **`GET /auth/whoami`** to see the current user and the **abilities of the current token**.

## Fleet service token (separate plane)

`/api/fleet/*` does **not** use Sanctum. Send:

```
Authorization: Bearer <PBX3_FLEET_SERVICE_TOKEN>
```

That secret is configured on the node (and matched on the control host). See [Endpoint reference — Fleet](reference.md#fleet-service-token).

---

## Auth requests

### Login

**POST /auth/login** — public

**Body**

```
'email' => 'required|email',
'password' => 'required',
'remember_me' => 'boolean',   # optional
```

**Response (200 OK)**  
`accessToken`, `token_type` (e.g. `"Bearer"`). Use `Authorization: Bearer <accessToken>` on later requests.

### Logout

**GET /auth/logout** — any Sanctum user

Revokes **the current token** only.

### Change own password

**PUT /auth/password** — any Sanctum user

**Body** (verify against `AuthController`): current password + new password fields as implemented.

### Whoami

**GET /auth/whoami** — any Sanctum user

Returns the authenticated user plus `abilities` for the **current token**.

### Register

**POST /auth/register** — `abilities:admin`

**Body**

```
'name' => 'required|string',
'email' => 'required|email|unique:users',
'password' => 'required|string',
'abilities' => 'array',          # optional; lexicon names only, e.g. ["admin"] or ["tenant","recordings"]
'endpoint' => 'nullable|numeric', # optional
```

**Response (201 Created)**  
Includes `accessToken` for the new user and assigned `abilities`.

---

## Users (admin only)

Require **`abilities:admin`**.

| Method | Path | Description |
|--------|------|-------------|
| GET | `/auth/users` | List users |
| GET | `/auth/users/{id}` | User by id |
| GET | `/auth/users/mail/{email}` | User(s) by email |
| GET | `/auth/users/name/{name}` | User(s) by name |
| GET | `/auth/users/endpoint/{endpoint}` | User(s) by SIP endpoint |
| PUT | `/auth/users/{id}` | Update user (abilities, endpoint, …) |
| PUT | `/auth/users/{id}/password` | Force-set password |
| DELETE | `/auth/users/revoke/{id}` | Revoke all tokens for user |
| DELETE | `/auth/users/{id}` | Delete user |

### Twin AstDB helpers (Sanctum + cluster validate)

| Method | Path |
|--------|------|
| PUT | `/auth/astamis/DBput/srktwin/{key}/{value}` |
| DELETE | `/auth/astamis/DBdel/srktwin/{key}` |

---

## Examples

### Login

```http
POST /api/auth/login
Content-Type: application/json

{"email":"admin@example.com","password":"secret"}
```

Save `accessToken` from the JSON response.

### Authenticated GET

```http
GET /api/extensions
Authorization: Bearer <accessToken>
```
