# Upgrade a package

## Same server, newer `.deb`

```bash
sudo apt install ./pbx3_*.deb
# and/or
sudo apt install ./pbx3api_*.deb   # if packaged that way for your lab
```

Postinst normalizes identity. Re-run `installer.sh` only if docs for that release say so.

## After upgrade checks

1. `curl …/up` still 200.
2. SPA login still works.
3. Top bar **Commit** if generator templates changed and panels show dirty.
4. Certificates still **in use** (Purchased / LE / Snakeoil).

## Empty identity in panel

```bash
sudo /opt/pbx3/scripts/normalize-globals-identity.sh
```

Do **not** run `reloader.sh` as an upgrade step.
