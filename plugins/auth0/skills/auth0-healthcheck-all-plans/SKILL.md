---
name: auth0-healthcheck-all-plans
description: Use when a customer or developer wants a complete, plan-aware health check of their own Auth0 tenant across ALL plans (Free, Essentials, Professional, Enterprise) — security & configuration hygiene PLUS use-case capability/plan fit in one pass. Runs one universal assessment, produces TWO scores (Security & Config Hygiene + Capability Fit), reframes recommendations for the tenant's current plan, recommends a specific self-service plan with cost OR an "Enterprise — contact sales" path with a prefilled sales brief when enterprise-only needs are detected, and can optionally apply approved fixes via the Auth0 CLI with per-command confirmation. Orchestrates the auth0-checkmate skill; emits an in-chat summary and, on request, a markdown + styled PDF report. Triggers include "health check my Auth0 tenant across all plans", "plan-aware Auth0 health check", "is my Auth0 tenant healthy and on the right plan", "full Auth0 tenant review Free to Enterprise", "should I be on Enterprise".
license: Apache-2.0
metadata:
  author: Carlos Aguilar <customeradvocate@auth0.com>
  version: '1.0.0'
  openclaw:
    emoji: "\U0001FA7A"
    homepage: https://github.com/auth0/agent-skills
    requires:
      bins:
        - auth0
    os:
      - darwin
      - linux
    install:
      - id: brew
        kind: brew
        package: auth0/auth0-cli/auth0
        bins: [auth0]
        label: 'Install Auth0 CLI (brew)'
---

# Auth0 Tenant Health Check (all plans)

You produce a **plan-agnostic** Auth0 tenant health check for any segment — Free, self-service paid (Essentials / Professional), or Enterprise. It fuses **CheckMate-style security/config hygiene** with **use-case capability fit**, gives **two scores**, makes **tier-adaptive** recommendations, and can **optionally apply approved fixes** via the Auth0 CLI.

This skill **orchestrates** the `auth0-checkmate` sister skill rather than reimplementing it: CheckMate runs the tenant scan + findings **and** gathers the lightweight company/use-case context (its Phase 3) that the Capability Fit track and the use-case-based plan recommendation build on.

References — open these when you need detail:
- [references/use-case-detection-logic.md](references/use-case-detection-logic.md) — classify B2B/B2C/AI/regulated (Phase 2)
- [references/feature-recommendations.md](references/feature-recommendations.md) — use-case → required features (Phase 3B)
- [references/production-readiness-checklist.md](references/production-readiness-checklist.md) — production-readiness = the summarized CheckMate report (Track A); plus Track B use-case capability + foundational items for the Fit score (Phase 3B)
- [references/feature-unlock-matrix.md](references/feature-unlock-matrix.md) — which features unlock per tier; Enterprise-only list (Phase 3B/4)
- [references/scoring-model.md](references/scoring-model.md) — the two-score definitions + bands (Phase 3)
- [references/plan-matching-logic.md](references/plan-matching-logic.md) — plan decision tree, with the Enterprise gate (Phase 4)
- [references/enterprise-need-detection.md](references/enterprise-need-detection.md) — when to route to "Enterprise — contact sales" (Phase 4)
- [references/mau-forecast-calculator.md](references/mau-forecast-calculator.md) — MAU runway vs. tier ceiling (Phase 4)
- [references/pricing.md](references/pricing.md) — **single source of truth** for all pricing, MAU limits, and feature availability
- [references/data-integrity-rules.md](references/data-integrity-rules.md) — input validation, confidence scoring, "Contact us" rule
- [references/report-template.md](references/report-template.md) / [references/report-template.html](references/report-template.html) — the two-section report (Phase 5)
- [references/talk-to-sales-block.md](references/talk-to-sales-block.md) — prefilled sales brief (Phase 4/5)
- [references/remediation-command-map.md](references/remediation-command-map.md) — gated-apply command shapes + safety (Phase 7)
- [references/checkmate-readme.md](references/checkmate-readme.md) / [references/auth0-cli-readme.md](references/auth0-cli-readme.md) — CheckMate run + CLI install/login
- [scripts/render_pdf.sh](scripts/render_pdf.sh) — HTML → PDF via headless Chrome (with fallbacks)

**Source-of-truth rules (non-negotiable):** all pricing / MAU limits / feature availability come from `pricing.md` — never hardcode them. `tenant_domain` always comes from the Auth0 CLI, never from the company context. Never quote or estimate a price for Enterprise.

**Company context is internal working context, not customer-facing output.** The customer is self-auditing their own tenant, so don't dump raw company research back at them. Surface only: (a) the derived **use-case classification**, (b) the customer's **product names** woven into the findings and recommendation, and (c) the **provenance / confidence note**. Don't echo back firmographics the customer already knows about themselves (company size, funding, and the like) — they add nothing and erode trust. The **use-case-based plan recommendation** (Phase 4) IS the customer's result — show it in full. Exception: the **Talk-to-Sales brief** may include company facts — it is addressed to sales, not shown as the customer's result.

---

## Workflow: Phases 0–7 (0–6 assess + report; 7 is opt-in apply)

### Phase 0 — Gather inputs (orchestrated; the user pre-stages nothing)

1. **Reviewer:** read `state/operator.json`. If missing, ask name + optional team/org. Blank org → the report subtitle is omitted (the common self-audit case). An optional `ae_link` may be stored for the Talk-to-Sales block.
2. **CheckMate (tenant facts + findings):** if the user has a CheckMate report (JSON), use it; otherwise **invoke the `auth0-checkmate` sister skill** to generate one (it bootstraps the CLI + M2M app and runs the audit; the user only completes the Auth0 device login). Parse findings from `data.report.summary[]` — each has `severity` (`info` | `warning` | `critical`), `title`, `severity_message`.
3. **Company context (company/use-case):** if provided, use it; otherwise it comes from the `auth0-checkmate` run — CheckMate's Phase 3 gathers lightweight public company context (business model, use case, products, login portals, third-party integrations) from the company domain. If CheckMate isn't run, gather the same context inline or ask the user. Record **provenance** (live research vs. training-knowledge fallback) and a **confidence** note.
4. **Normalize the fact set** (from whichever source supplied each): `custom_domains_present`, `log_streams_present`, `mfa_configured`, `organizations_configured`, `enterprise_connections_count`, `applications_count`, `current_plan` (fallback "Free"), `current_mau`, `monthly_growth_rate`, `tenant_domain` (always from CLI), plus the CheckMate severity buckets. Record the **source** of each (scan vs. user-supplied) for confidence reporting.
5. **MAU/growth (use every source, in priority order):** (a) **authoritative** — `auth0 api get stats/active-users` for current MAU + `stats/daily` for a real growth trend (needs `read:stats`; add the scope if it 403s); (b) **tenant telemetry / provided data** — any current-MAU or growth figures already supplied or cached; (c) **company context as context only** — growth-stage/scale hints to sanity-check and set growth expectations, but a company's product-user count is NOT its Auth0 tenant MAU, so never substitute it for the actual active-users figure; (d) **ask** the user for current MAU + expected growth (presets; default 15%, labeled) — don't send them to export a CSV; only point to the Support Center Quota/Usage report (online, up to 12 months, no CSV) if they ask where to find it. **Compute actual growth from history when available** rather than assuming. Full order in [references/mau-forecast-calculator.md](references/mau-forecast-calculator.md).

See **Graceful degradation** at the end for missing CheckMate / missing company context.

### Phase 1 — Auth0 CLI bootstrap (only when self-driving the CLI)

Needed when this skill drives the CLI directly (gathering extra facts in Phase 0, or applying fixes in Phase 7). Skip if the assessment runs purely off a provided CheckMate JSON and the user declines the apply step.
```bash
auth0 --version
auth0 tenants list --json
```
Install if missing (per-platform commands in [references/auth0-cli-readme.md](references/auth0-cli-readme.md)); `auth0 login --scopes "create:client_grants,read:client_grants"` if empty/401. Pin `tenant_domain` from `tenants list --json` (don't parse region from the suffix).

### Phase 2 — Classify use case

Run the [references/use-case-detection-logic.md](references/use-case-detection-logic.md) decision tree (regulated vertical first → B2B/B2C/Mixed → AI). Output: `detected_use_case`, `verticals[]`, `business_model`, `ai_use_case`, AI integrations.

### Phase 3 — Two-part assessment + TWO scores

One universal assessment, two scored tracks (full definitions + bands in [references/scoring-model.md](references/scoring-model.md)).

**3A — Security & Config Hygiene (universal, plan-independent).** From CheckMate `summary[]`: count by severity, compute the **Hygiene Score**. This is a security claim — **if there was no CheckMate scan, do NOT emit a number** ("Not scored — run a CheckMate audit"); mark low-confidence. Output feeds Part A + Phase 7 Loop A.

**3B — Capability Fit (tier-aware framing, plan-independent number).** Build the feature-gap matrix: required features for the use case ([references/feature-recommendations.md](references/feature-recommendations.md), including the use-case capability + FOUNDATIONAL items from [references/production-readiness-checklist.md](references/production-readiness-checklist.md) so the score stays graduated) vs. configured, marked ✅ / ❌ / ⚠️. (Production-readiness itself = Part A / CheckMate; Part B is the use-case *expansion* layer.) Tag each gap's **Plan Home** via [references/feature-unlock-matrix.md](references/feature-unlock-matrix.md) + [references/pricing.md](references/pricing.md): *Available now on `current_plan`* / *Unlocks on `<plan>`* / *Enterprise-only*. Compute the **Capability Fit Score**. The number is the same on any plan; only the framing/remediation path differs.

### Phase 4 — Tier-adaptive recommendation

1. **MAU forecast** vs. the tenant's own track ceiling — [references/mau-forecast-calculator.md](references/mau-forecast-calculator.md) + [references/pricing.md](references/pricing.md). Never hardcode limits.
2. **Enterprise-need detection FIRST** — run [references/enterprise-need-detection.md](references/enterprise-need-detection.md). It can short-circuit plan matching.
3. **Plan matching** keyed on current plan — [references/plan-matching-logic.md](references/plan-matching-logic.md):
   - **Self-service (no Enterprise need):** recommend a specific plan + **exact cost** from `pricing.md`. Suggest the **A4AA add-on only when `a4aa_fit_score ≥ 0.4` AND there are concrete agent integrations** (`integrations.length > 0` / autonomous-action workflows in the company context) — a high fit score alone, with no integrations, is NOT enough (per `plan-matching-logic.md`). The `a4aa_fit_score` here is the health check's own metric, computed from use-case + integration signals (per `data-integrity-rules.md`), not an external score.
   - **Enterprise need = TRUE:** recommend **"Enterprise — contact sales"** with **NO price**, and emit the Talk-to-Sales block ([references/talk-to-sales-block.md](references/talk-to-sales-block.md)).
   - **Enterprise need = SOFT:** recommend the best self-service plan + cost, AND add a "you may also qualify for Enterprise" note + offer the Talk-to-Sales block.
   - **Already on Enterprise:** no upsell — optimization / governance / feature-adoption of what they already own.

### Phase 5 — Output (chat always; md/PDF on request)

**Always — in-chat layered summary:**
1. **What I checked** — tenant, current plan, data provenance.
2. **Two headline scores** — `Hygiene NN/100 — <band>` and `Capability Fit NN/100 — <band>` (each with confidence). Hygiene shows "Not scored" if there was no scan.
3. **Top 3–5 findings** across both tracks, each with a one-line **"why this matters for `<Company>`"** in the customer's product terms.
4. **Recommendation line** — self-service plan + cost, or "Enterprise — contact sales", or Enterprise optimization. **Render the next step as a clickable markdown link in chat, never a bare URL:** self-service upgrade → `[Upgrade in the Auth0 Dashboard](https://manage.auth0.com/dashboard/<region>/<tenant>/billing)` (build the URL from the pinned tenant; fall back to `[Auth0 Dashboard](https://manage.auth0.com/)` if region/tenant aren't known); Enterprise / sales → `[Contact Auth0 sales](https://auth0.com/contact-us)` (or the custom AE link from `state/operator.json.ae_link` if set). Documentation references should likewise be `[label](url)` links.
5. **Confidence / provenance note.**

**On request ("generate the report" / "PDF"):** produce three files with one timestamped basename in `~/Documents/` (fallback `~/auth0-healthcheck-reports/`):
```
auth0_healthcheck_<sanitized_tenant>_<YYYYMMDD_HHMMSS>.{md,html,pdf}
```
Markdown per [references/report-template.md](references/report-template.md); HTML per [references/report-template.html](references/report-template.html); PDF via:
```bash
${CLAUDE_SKILL_DIR}/scripts/render_pdf.sh "$HTML_PATH" "$PDF_PATH"
```
If the renderer exits non-zero, surface its stderr — the md + html are already saved (don't fail the run).

**Fusion lint before saving (must be empty):**
```bash
grep -nE '\{\{|the customer[^A-Za-z]|enterprise clients[^A-Za-z]|the affected apps' "$MD_PATH" "$HTML_PATH"
```
Every gap/opportunity must name a real product, app, or segment. No `{{placeholder}}` may survive. Prices appear only for self-service recommendations — never for Enterprise.

### Phase 6 — Walk the user through it

Render the chat summary, give the file paths, and offer Phase 7 for the current-plan-achievable items.

### Phase 7 — Optional gated apply (opt-in)

Enter only on explicit opt-in. Use [references/remediation-command-map.md](references/remediation-command-map.md) for command shapes, the MFA-lockout safety rule, fix-dependency ordering, and the never-without-confirmation list. Per item: build → **show diff** (current vs. proposed via `auth0 api get`) → print exact command(s) → `AskUserQuestion` (Implement now / Queue / Skip) → execute → **verify by re-fetch** → never destructive retry.

- **Loop A** — "Immediate Actions — Available Today" items.
- **Gate** — ask whether they've upgraded. Not yet → queue `pending_upgrade` + give the clickable billing link `[Upgrade in the Auth0 Dashboard](https://manage.auth0.com/dashboard/<region>/<tenant>/billing)`. **If the recommendation was "Enterprise — contact sales", there is no self-service gate** → surface the Talk-to-Sales block and queue plan-gated items as `pending_enterprise`.
- **Loop B** — "After Upgrading" items (only once upgraded).

Close out: update state, append to `history.jsonl`, print applied/queued/skipped counts, suggest re-running the health check to confirm fixes.

State dir: `~/.auth0-checkmate/state/` (`operator.json`, `setup.json` (cache; secrets excluded), `enrichment_<domain>_<ts>.json`, `queue.json`, `history.jsonl`). Treat `setup.json` as a cache — re-validate each run.

---

## Graceful degradation & confidence

- **No CheckMate scan:** offer to invoke `auth0-checkmate`; if declined / no Auth0 access, ask the tenant facts in plain language ("Do users log in at your own domain like `login.yourco.com`?", "Is MFA turned on?", "How many enterprise SSO connections?") with an "I don't know" option that defaults to *not configured*; mark **low-confidence**. **Hygiene is NOT scored** in this mode; add to the output: *"Based on user-supplied configuration; this is not a security audit. For scored security findings, run the auth0-checkmate skill."*
- **Missing / low-confidence company context:** run a technical-only assessment; ask `business_model` if null; add *"Company context unavailable; recommendation based on technical posture alone."* If the context came from a training-knowledge fallback (low confidence), treat company specifics as approximate, **downgrade an inferred (Class B) Enterprise recommendation to SOFT**, and add *"Company details are drawn from general knowledge and may be out of date; verify specifics before sharing externally."*
- Every score and the recommendation report their own confidence (0–1) and source; the report footer and chat both carry the confidence note.

## Pitfalls to remember

- `pricing.md` is the only source for prices, MAU limits, and feature availability — never hardcode, never let a reference file contradict it.
- `tenant_domain` is always the CLI value, never derived from the company context or the company domain.
- Enabling an MFA factor ≠ enforcing it — verify a factor can deliver before enforcing (see `remediation-command-map.md`).
- Never quote or estimate an Enterprise price.
- Don't reinvent CheckMate — invoke it (it also supplies the company context).
