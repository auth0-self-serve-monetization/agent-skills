# Scoring Model — Two Independent Scores

This skill emits **two separate 0–100 scores**, never blended into one number. Each mirrors a section of the report:

- **Security & Config Hygiene** → Part A (universal, plan-independent)
- **Capability Fit** → Part B (tier-aware framing, plan-independent number)

Each score is reported with its own **confidence** (0.0–1.0, per [data-integrity-rules.md](data-integrity-rules.md)) and its band label.

---

## Score 1 — Security & Config Hygiene (universal)

Derived **only** from CheckMate findings (`data.report.summary[]`). Plan-independent — security hygiene is the same expectation on Free or Enterprise. Weighted deduction so severity dominates raw count:

```
weighted_failures = 5·(#critical) + 2·(#warning) + 0.5·(#info_failing)
weighted_total    = weighted_failures + 1·(#passing)
penalty_ratio     = clamp(weighted_failures / weighted_total, 0, 1)
hygiene_score     = round(100 × (1 − penalty_ratio))
```

- All checks passing → 100.
- **No CheckMate scan available → DO NOT emit a number.** Show `Hygiene: Not scored — run a CheckMate audit for a security score` and mark low-confidence. Hygiene is a security claim; never fabricate it from self-reported config.
- **Passing-count fallback.** The formula needs `#passing`, but CheckMate's `summary[]` may not expose a passing count explicitly (the pass/fail field shape is unconfirmed — verify against a fresh report). If a passing count is **not** available, derive the denominator from the total number of checks instead: `weighted_total = weighted_failures + (total_checks − #failures)`, using the count of `summary[]` entries (or CheckMate's documented ~50-check total) as `total_checks`. If neither failures-with-total nor a passing count can be established, fall back to **"Not scored"** rather than guessing.

**Bands:** 90–100 Excellent · 75–89 Healthy · 50–74 Needs Attention · 25–49 At Risk · 0–24 Critical.

---

## Score 2 — Capability Fit (tier-aware framing, plan-independent number)

From the Phase 3B feature-gap matrix, weighted by each required feature's gap severity (CRITICAL/HIGH/MODERATE/LOW from [use-case-detection-logic.md](use-case-detection-logic.md)):

```
weight: CRITICAL = 4, HIGH = 3, MODERATE = 2, LOW = 1
required_weight   = Σ weight(required_feature)
configured_weight = Σ weight(✅ configured feature)   ;  ⚠️ partial counts at 0.5 × weight
fit_score         = round(100 × (configured_weight / required_weight))
```

**Key rule — the number is plan-independent; only the framing is tier-aware.** A missing required feature deducts whether it's a free toggle, a paid unlock, or Enterprise-only. The "available now / unlocks on Plan X / Enterprise-only" distinction lives in the gap matrix's **Plan Home** column and the recommendation, NOT in the score. So the same tenant gets the same Fit score on any plan, but a different remediation path. Missing Enterprise-only required features still deduct — that deduction is exactly what surfaces an Enterprise recommendation.

**Bands:** 80–100 Ready · 60–79 Mostly Ready · 40–59 Partially Ready · 0–39 Not Ready.

**Avoid the all-or-nothing zero.** If the required-feature set for a use case is *only* the advanced/enterprise capabilities, a fresh Free tenant scores 0 every time — low signal, and easily misread as "the tenant is broken." Two rules to keep the score graduated and honest:
1. The required set per use case (in [feature-recommendations.md](feature-recommendations.md)) **must include the foundational capabilities** that a working tenant already has — a configured primary connection, session/cookie config, basic branding, a verified email/social login path — each at LOW weight. These are usually ✅, so a functioning-but-unspecialized tenant lands in a meaningful low-but-nonzero range rather than 0.
2. Use **⚠️ partial (0.5×)** generously where a feature is present but not fully configured (e.g. MFA enabled but not enforced; one social connection of several).
3. When Fit is genuinely 0 or near-0, the output MUST frame it as *"not yet set up for `<use_case>` — here's what to configure,"* never as a defect. A low Fit on Free is expected and is the call-to-action, not a failure grade.

---

## Confidence

Both scores carry a confidence value (per [data-integrity-rules.md](data-integrity-rules.md)):
- 0.9–1.0 — all inputs verified (real CheckMate scan + live enrichment)
- 0.5–0.8 — some inputs inferred or partially supplied
- < 0.5 — mostly user-supplied / training-knowledge fallback → surface prominently; Hygiene is "Not scored" in this case.

Display format: `Hygiene 82/100 — Healthy (confidence 0.9)` · `Capability Fit 55/100 — Partially Ready (confidence 0.7)`.

> Weights (5/2/0.5 and 4/3/2/1) are v1 proposals. Sanity-check the bands against 2–3 known tenants and tune before treating the numbers as authoritative.
