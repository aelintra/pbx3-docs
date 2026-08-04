# Dial prefixes — convert from InterSARK / INTERSITE

Operators converting SARK-era sister-site digit maps (InterSARK / SailToSail trunks, `*_INTERSITE` OutRoutes) into pbx3 **Dial prefixes** should follow the full recipe in the repository:

**Workingdocs (source of truth):** `pbx3/workingdocs/DIAL_PREFIX_LEGACY_MIGRATE.md`

## Short form

1. Inventory InterSARK / INTERSITE per **calling** tenant.  
2. Pick non-colliding 2–4 digit prefixes → **tenant FQDN** of the sister site (not the instance hostname).  
3. SPA **Outbound → Dial prefixes** as instance admin; genAst on each home.  
4. **Dual-run** with legacy until prefix dial is proven.  
5. Retire INTERSITE / InterSARK **only** after training — never auto-delete PSTN routes.

Product design: `pbx3/workingdocs/TENANT_SHORT_DIAL_REQUIREMENTS.md`.
