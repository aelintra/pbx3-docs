# Dial prefixes — convert from legacy sister-site maps

!!! warning "Not the product short-dial model"
    **Site Groups** are the supported fleet operator model for sister-site short dial.  
    See [Site Groups](site-groups.md).

    This page is only for converting **previous-PBX sister-site digit maps** (legacy trunk techs `InterSARK` / `SailToSail`, `*_INTERSITE` OutRoutes) into **interim** instance dial-prefix rows (lab / dual-run). It is **not** a migrate into Site Groups, and **not** guidance to invent production meshes by hand.

Operators converting those maps into pbx3 **Dial prefixes** should follow the full recipe in the repository:

**Workingdocs (source of truth):** `pbx3/workingdocs/DIAL_PREFIX_LEGACY_MIGRATE.md`

## Short form

1. Inventory legacy sister-site trunks / `*_INTERSITE` OutRoutes per **calling** tenant.  
2. Pick non-colliding 2–4 digit prefixes → **tenant FQDN** of the sister site (not the instance hostname).  
3. SPA deep-link `/dialaliases` as instance admin (panel not in sidebar); genAst on each home.  
4. **Dual-run** with legacy until prefix dial is proven.  
5. Retire INTERSITE / legacy sister-site trunks **only** after training — never auto-delete PSTN routes.

When Site Groups are enabled for the fleet, move sister-site membership and routing prefixes to **Fleet → Site Groups** instead of maintaining hand meshes. There is no automatic import from hand-invented wild meshes.

Product design (call path): `pbx3/workingdocs/TENANT_SHORT_DIAL_REQUIREMENTS.md`.  
Site Group / release model: `pbx3/workingdocs/DIAL_COHORT_REQUIREMENTS.md`.
