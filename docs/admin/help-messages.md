# Help messages

Field hints come from help tables (e.g. `tt_help_core`).

After restore, if hints are empty, merge seeds:

```bash
sudo sqlite3 /opt/pbx3/db/sqlite.db < /opt/pbx3/db/db_sql/sqlite_message.sql
```

Editing help text in-product (when exposed) is for operators documenting your lab — changes are data, not package updates.
