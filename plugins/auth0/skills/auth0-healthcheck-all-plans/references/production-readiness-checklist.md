# Production-Readiness Grounding

**"Production readiness" = a summarized view of the CheckMate report.** CheckMate's ~50 checks ARE the authoritative production-readiness checklist — there is no separate document to source. So **Part A (Security & Config Hygiene) is already the production-readiness assessment**: it reads directly from CheckMate findings and presents them as the prioritized, production-readiness summary.

This file's job is therefore narrow:
1. Map what CheckMate covers, so Part A can render a clean **production-readiness summary** (not invent its own criteria).
2. Supply the **Part B (Capability Fit)** use-case items + **foundational** items that keep the Fit score graduated (see [scoring-model.md](scoring-model.md)). Part B is the *use-case expansion* layer, distinct from production-readiness.

Each item notes **Track** (A = production-readiness via CheckMate · B = use-case capability) and a default **severity** (CRITICAL/HIGH/MODERATE/LOW/FOUNDATIONAL) for Fit weighting.

## Foundational (Track B, FOUNDATIONAL weight) — most working tenants already have these

Include these in every use case's required set so a functioning Free tenant doesn't score 0 Fit:
- A configured primary connection (Database or primary Social) — login works
- Reasonable session/cookie configuration
- Basic tenant branding (logo/colors) on the login page
- At least one verified login path (email/password or social)

## Production readiness (Track A — the CheckMate report, summarized)

CheckMate is the source of truth for these; this is a human-readable map of what it validates, so Part A can summarize it:
- Custom domain configured for production (no `*.auth0.com` in prod)
- MFA available **and enforced** appropriately for the audience
- Attack protection on: brute-force, breached-password detection, suspicious-IP throttling
- Grant-type hygiene: no implicit grant / ROPG where avoidable; short token lifetimes
- No dev/test callback URLs or origins on production apps
- Log streaming to a SIEM / monitoring destination
- Branded email provider (no Auth0-default emails in prod)
- Separate dev / staging / prod tenants
- Secrets not exposed client-side; M2M secrets scoped and rotated
- Unused connections disabled; DB connection security reviewed

> If a CheckMate finding isn't represented above, **CheckMate wins** — this map is descriptive, not authoritative. Keep it in sync with the CheckMate check set, not the other way around.

## Use-case capability (Track B — feeds the gap matrix)

- **B2B SaaS:** Organizations CRITICAL · Enterprise Connections / SSO CRITICAL · RBAC HIGH · per-org branding MODERATE
- **B2C:** social connections breadth HIGH · passwordless/passkeys MODERATE · custom domain HIGH · sign-up/login UX MODERATE
- **AI / agents:** Token Vault (when integrations exist) HIGH · CIBA for high-stakes autonomous actions HIGH · M2M auth MODERATE
- **Regulated vertical:** log streaming CRITICAL · breached-password detection HIGH · MFA enforcement CRITICAL · (compliance add-ons like HIPAA/BAA are **Enterprise-only** → route via [enterprise-need-detection.md](enterprise-need-detection.md))
