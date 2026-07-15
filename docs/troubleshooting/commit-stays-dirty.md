# Commit stays dirty

## Normal case

Panel **Save** writes the DB. Top bar stays dirty until **Commit** regenerates Asterisk and reloads.

If phones unchanged: you probably forgot Commit.

## Extensions empty in Asterisk after Commit

Once (legacy lab symptom):

```bash
sudo php /opt/pbx3/php/utilities/runLinker.php
```

Then Commit again. Symlink `/etc/asterisk` → `/opt/pbx3/etc/asterisk/configs/` should exist on packages **0.0.3-22+**.

## Generator / template errors

SSH to the node and inspect API / generator logs after a failed Commit; fix the reported panel/table, Save, Commit again.
