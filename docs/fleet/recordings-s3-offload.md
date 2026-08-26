# Recordings S3 offload (S7) — capability + tenant opt-in

**Audience:** fleet operators. Call **recording** (MixMonitor / who gets recorded) is separate — this page is **whether archived wavs are copied to the fleet recordings object store**.

**Product lock (2026-08-26):** two layers — see **`RECORDINGS_STORAGE_DESIGN.md`** § Capability vs policy.

| Layer | Control | Default |
|-------|---------|---------|
| **Capability** | Ask at **control install** (and wire home plumbing). Off → no recordings bucket / no home upload. | **Off** |
| **Policy** | Tenant field **`rec_s3`** (`YES`/`NO`) on Call recording | **`NO`** |

Home `/opt/pbx3api/.env` is **plumbing** (`PBX3_RECORDING_UPLOAD_ENABLED` + gatekeeper URL/token), not the product opt-in. Fleet → Instances may later show **read-only** plumbing health — not the tenant switch.

## What “on” means

| Layer | Role |
|-------|------|
| Home disk | Spool → local archive + SQLite `recordings` rows (R1 / R1.5) — works without S3 |
| Capability | Dedicated recordings bucket + gatekeeper `PBX3_RECORDINGS_BUCKET` + home upload cron/env ready |
| Tenant `rec_s3=YES` | `pbx3:recordings-s3-upload` PUTs that tenant’s rows via gatekeeper presign |
| SPA Recordings | One list (local + S3-backed). **S3** badges after upload has run for opted-in tenants |

**Fail-safe:** telephony does not depend on S7. Gatekeeper or object store down → calls continue; local archive remains authoritative until upload succeeds.

## Install (capability)

At **control** install, answer **Enable call recordings S3 offload?** (default **No**).

- **No** — org/catalog bucket only; homes stay local-only for recordings.
- **Yes** — create `{stem}-pbx3-recordings` (AWS: `create-recordings-bucket.sh`; Garage: dedicated bucket + key allow R/W). Set `PBX3_RECORDINGS_BUCKET`. **Lab Garage:** bind S3 API so homes can reach it (`0.0.0.0:3900` or control LAN IP) and set `AWS_ENDPOINT=http://<control-lan-ip>:3900` (not `127.0.0.1`).

**Home** (when capability Yes): set upload plumbing — `PBX3_RECORDING_UPLOAD_ENABLED=true`, `PBX3_GATEKEEPER_URL`, `PBX3_GATEKEEPER_TOKEN` (= control **`GATEKEEPER_API_TOKEN`** / `fleet_admin`, **not** the fleet service token), lab `PBX3_GATEKEEPER_HTTP_VERIFY=false`, and keep `/etc/cron.d/pbx3-recordings`.

Solo (no fleet): no recordings bucket; local retention only.

## Tenant policy (`rec_s3`)

On the instance SPA: **Tenants → Call recording → S3 offload** (`rec_s3`). Default **No**.

Upload runs only when **capability is on** on that home **and** `rec_s3=YES` for the tenant. Turning `rec_s3` off stops new uploads; existing S3 objects / catalog `s3_key` rows remain until retention.

`PBX3_RECORDING_UPLOAD_TENANTS` in home `.env` is **break-glass** only (further restrict which tenants may upload); it is not the primary product control.

## Manual plumbing (existing fleets / break-glass)

If install did not enable capability, ops can still wire it later:

1. Create recordings bucket; set control `PBX3_RECORDINGS_BUCKET`.
2. On home `.env`:

```bash
PBX3_RECORDING_UPLOAD_ENABLED=true
PBX3_GATEKEEPER_URL=https://control.example.com   # lab: http://192.168.1.33
PBX3_GATEKEEPER_TOKEN=<control GATEKEEPER_API_TOKEN — break-glass / fleet_admin>
# PBX3_GATEKEEPER_HTTP_VERIFY=false   # lab HTTP
```

3. Set tenant `rec_s3=YES` for sites that should offload.
4. `sudo -u www-data php artisan config:clear` then `pbx3:recordings-s3-upload`.

To disable **capability** on a home: `PBX3_RECORDING_UPLOAD_ENABLED=false` (upload no-ops for all tenants).

## Verify

1. Capability on; one tenant `rec_s3=YES`; place a recorded call; wait for local offload.
2. Run `pbx3:recordings-s3-upload` (or wait for cron).
3. SPA **Recordings** → Storage **Local + S3** (then **S3 only** after local age-out).
4. A tenant with `rec_s3=NO` stays local-only in the same list.

## Related

| Doc | Use |
|-----|-----|
| Product **`RECORDINGS_STORAGE_DESIGN.md`** | Capability vs policy lock |
| **`CONTROL_HOST.md`** | Gatekeeper + `PBX3_RECORDINGS_BUCKET` |
| **`OPS_S3_RUNBOOK.md`** §13 / §13.5 | AWS bucket + enable notes |
| Cron example | `pbx3api/scripts/cron.d/pbx3-recordings.example` |
| Instance SPA | Tenant Call recording (`rec_s3`); ACD → **Recordings** (listen/filter) |
