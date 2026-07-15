# SPA catalog URL and Pages CORS

## Production model

- Central **pbx3spa** on **GitHub Pages** (custom domain optional)
- Instances remain **API-only** (`:44300`)
- SPA build sets `VITE_INSTANCE_DIRECTORY_URL` → `https://…/catalog/instance-index.json`

## Checklist

1. Bucket CORS: Pages origin(s) (+ localhost for lab).
2. **Each** node API CORS: same SPA origin + `Authorization`.
3. Solo trial: omit catalog URL ([Rule 6](../getting-started/solo-trial.md)).
4. Lab Vite may proxy `/dev-catalog` to S3 to bypass browser CORS — that is **dev-only**.

## Lab catalog

`https://08jzwn-pbx3.s3.us-east-1.amazonaws.com/catalog/instance-index.json`
