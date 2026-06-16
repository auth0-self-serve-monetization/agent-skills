# Enterprise-Need Detection

Decides whether to route a tenant to **"Enterprise — contact sales"** instead of a self-service plan. Runs in Phase 4 **before** plan matching, and can short-circuit it.

There is **no explicit "enterprise need" field** in enrichment output — it must be synthesized from enrichment fields + the Phase 3B feature-gap matrix + explicit user asks.

The authoritative list of Enterprise-only / "Contact us" capabilities comes from [feature-unlock-matrix.md](feature-unlock-matrix.md) and [pricing.md](pricing.md). Keep this file in sync with those — do not invent gating that contradicts them.

---

## Class A — Explicit (any one → Enterprise, HIGH confidence)

```
IF the user explicitly asks for an Enterprise-only feature,
   OR a REQUIRED/MISSING feature in the Phase 3B gap matrix is Enterprise-only:
   {
     HIPAA / BAA,
     Bot Detection,
     Credential Guard,
     Adaptive MFA,
     FAPI-certified Security Profile,
     Tenant Access Control Lists (ACLs),
     Continuous Session Protection,
     Private deployment,
     custom rate limits,
     99.99% SLA,
     Home Realm Discovery (B2C),
     M2M Access for Organizations (B2C),
     MAU beyond the published self-service ceiling for the tenant's track
   }
THEN enterprise_need = TRUE
     trigger        = "explicit feature: <name>"
     confidence     = HIGH
```

Class A short-circuits immediately — no need to evaluate Class B.

---

## Class B — Inferred (synthesis; each matched signal = 1 point)

```
S1  industry ∈ regulated { fintech, banking, payments, healthcare, insurance, government }   [regulated]
S2  employee_count_range > 1000                                                              [scale]
S3  enterprise-scale valuation/revenue (e.g. latest_valuation > $1B) OR is_public == true    [scale]
S4  login_portal_assessment names 4+ distinct enterprise portals / corporate IdPs            [SSO surface]
S5  a4aa_fit_score > 0.6                                                                      [advanced AI]
S6  enterprise_connections needed or in use ≥ 6 (beyond B2B Professional's 5 included)        [SSO volume]
S7  MAU forecast crosses the tenant's track published ceiling within 12 months               [capacity]

points = S1 + S2 + S3 + S4 + S5 + S6 + S7

points >= 2 → enterprise_need = TRUE
              confidence = MEDIUM..HIGH (rises with points)
              trigger    = list of matched signals
points == 1 → enterprise_need = SOFT
              → recommend best self-service plan + "you may also qualify for Enterprise" note
              → offer the Talk-to-Sales block
points == 0 AND Class A not hit → enterprise_need = FALSE → self-service plan matching
```

---

## Overrides & guardrails

- **Already on Enterprise** → skip detection entirely; go to the optimize/governance branch (no upsell).
- **Never quote or estimate a price** when `enterprise_need = TRUE` — output "Enterprise — contact sales" only (enforced by [data-integrity-rules.md](data-integrity-rules.md) "Contact us" rule).
- **Confidence gating:** if enrichment came from a training-knowledge fallback, or enrichment `confidence_score < 0.5`, **downgrade an inferred (Class B) Enterprise call from TRUE to SOFT** and say so in the output. Class A (explicit user ask / hard feature requirement) is NOT downgraded.
- When `enterprise_need` is TRUE or SOFT, populate the Talk-to-Sales block (see [talk-to-sales-block.md](talk-to-sales-block.md)) with the matched triggers as the "why now."

---

## Output object

```json
{
  "enterprise_need": "TRUE | SOFT | FALSE",
  "confidence": "HIGH | MEDIUM | LOW",
  "class": "A | B | none",
  "triggers": ["regulated industry", "employees > 1000", "explicit feature: HIPAA/BAA"],
  "enterprise_features_needed": ["HIPAA/BAA", "Adaptive MFA"]
}
```
