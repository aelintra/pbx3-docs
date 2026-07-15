# pbx3-docs

Published **installer and administrator** documentation for PBX3 (MkDocs Material).

Not developer workingdocs — those stay in the product repos under `workingdocs/`.

## Local review

```bash
cd pbx3-docs
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
mkdocs serve
```

Open http://127.0.0.1:8000/

## GitHub Pages

Remote: **https://github.com/aelintra/pbx3-docs** (interim until the PBX3 OSS org exists).

Push to `main` with Actions enabled; workflow runs `mkdocs gh-deploy --force` (same pattern as sail6-docs). Published site: **https://aelintra.github.io/pbx3-docs/**.

## Content map

Nav and page inventory: `pbx3/workingdocs/USER_GUIDES_MKDOCS_CONTENT_MAP.md`.
