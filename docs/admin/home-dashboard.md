# Home dashboard

Route: `/`

## Status

- PBX **Running** / **Stopped**
- Actions: **Start**, **Stop**, **Reboot** (use carefully on live tenants)

## Commit (top bar)

- Red / dirty = saved panel changes not yet generated into Asterisk
- **Commit** regenerates config and reloads services
- **Save ≠ Commit** — phones do not change until Commit

Lab habit: after every config edit session, glance at the top bar before testing a handset.
