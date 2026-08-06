# Methods and notation

## HTTP methods

pbx3api uses **GET**, **POST**, **PUT**, and **DELETE**.

| Method | Typical use |
|--------|-------------|
| **GET** | Retrieval. Some GETs are operational (commit, reboot, take backup, restart firewall). |
| **POST** | Create a new instance, or upload a file (multipart). |
| **PUT** | Update an existing instance; also restore backup/snapshot, restart firewall, etc. |
| **DELETE** | Destroy an instance |

**Notes:**

- Regular JSON responses are JSON objects/arrays
- Regular POST/PUT bodies must be **raw JSON** (`Content-Type: application/json`)
- File upload (backups, greetings, snapshots, custom certs) uses **multipart form** bodies

## Request notation

An HTTPS request looks like:

```
METHOD /{resource}/{id?}/{parameter?}
```

A trailing `?` on a path segment means optional. Example:

```
GET /agents
GET /agents/{agent}
```

The first returns the list; the second returns one agent (route binding by shortuid / id / pkey as implemented).

POST/PUT bodies in the [endpoint reference](reference.md) are described with **Laravel validation** rule strings, for example:

```
'pkey' => 'required|integer|min:1000|max:9999',
'cluster' => 'required|exists:cluster,pkey',
'name' => 'required|alpha_dash',
```

Laravel validation rules are documented at [laravel.com/docs/validation](https://laravel.com/docs/11.x/validation#available-validation-rules).

- **POST:** required fields are the minimum to create; you may also send other updateable fields
- **PUT:** listed fields are updateable; omit fields you do not change
- When bodies here are marked *verify*, check the controller `$updateableColumns` or **`GET /schemas`**

## Ability gates (summary)

| Gate | Examples |
|------|----------|
| Public | `POST /auth/login` |
| Any Sanctum user | logout, whoami, change own password, `GET /fleet-posture` |
| `ability:admin,tenant` | Most tenant ops, schemas, commit, CDR, greetings read/write, … |
| `ability:admin,recordings` | Recordings list / stream / download |
| `abilities:admin` | Users, trunks, routes, tenants, AMI, backups, firewall, certificates, dialaliases, … |
| `fleet.token` | `/api/fleet/*` |

## HTTP status codes

Standard HTTP status codes apply. Validation and application errors often return JSON error payloads (e.g. `message`, `Error`, or Laravel validation `errors`).

See also [httpstatuscodes.com](https://www.restapitutorial.com/httpstatuscodes.html).

## Related control planes

| API | Audience | Docs |
|-----|----------|------|
| Instance pbx3api (`:44300/api`) | This section | You are here |
| Gatekeeper fleet UI API | Operators / SPA Fleet mode | Fleet ops guides under **Fleet operations** — not this digest |
| SBC admin API | Edge | Fleet / SBC install guides |
