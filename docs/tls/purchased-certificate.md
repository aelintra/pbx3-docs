# Purchased (custom) certificate

## Install

1. SPA → **Certificates** → Purchased.
2. **Install** fullchain PEM + private key.
3. Top line: **Currently in use: Purchased**.

Purchased **overrides** Let's Encrypt while custom files exist.

## Remove

**Remove** → falls back to LE (if present) or snakeoil; nginx/Asterisk reload.

## Notes

- One custom path only (wildcard / multi-SAN commercial certs are fine).
- Include `/opt/pbx3/etc/ssl/custom/` and `le-domain` in backup expectations.
- Manufacturer device certs (3rd-party panel) are separate from this page.
