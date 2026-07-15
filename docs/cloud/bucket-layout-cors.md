# Org bucket layout and CORS

AWS S3 is the **reference** implementation. S3-compatible object stores can work if the gatekeeper/node adapters agree — product rules do not hard-wire “must be AWS”.

## Naming

`{shortid}-pbx3` — **no dots**. Lab: **`08jzwn-pbx3`**, region `us-east-1`.

One org bucket for the whole fleet — not one bucket per node.

## Prefixes

| Prefix | Who | Public? |
|--------|-----|---------|
| `catalog/*` | SPA GET; registrar/control PUT | Anonymous **read** only |
| `instances/{ksuid}/*` | Node role for own backups | Private |
| `tenants/*` | Control / registrar | Private |

## Create sketch (console)

1. Create bucket → allow public **read only** where catalog needs it (tighten BPA carefully).
2. Bucket policy: `s3:GetObject` on `catalog/*` only for public/anonymous.
3. Upload `catalog/instance-index.json`.
4. Verify: curl catalog → JSON; `instances/…` anonymous → **403**.

## CORS (SPA + Pages)

`AllowedOrigins` must include the SPA origin (GitHub Pages and/or `http://localhost:5173` for lab Vite). Methods at least `GET`, `HEAD`.

See also [SPA catalog URL and Pages CORS](spa-catalog-cors.md).
