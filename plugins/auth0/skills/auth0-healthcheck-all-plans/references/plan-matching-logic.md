# Plan Matching Logic

> **Used by `auth0-healthcheck-all-plans` (plan-agnostic).** Maps current plan + use case + gaps → a recommended plan. Runs AFTER the Enterprise-need gate below.

## Step 0 — Enterprise-need gate (run FIRST)

Before any self-service matching, run [enterprise-need-detection.md](enterprise-need-detection.md):

- **Enterprise need = TRUE** (an Enterprise-only feature is asked for / required, or ≥2 inferred signals) → **short-circuit: recommend "Enterprise — contact sales" with NO price**, emit the Talk-to-Sales block, SKIP self-service matching, and queue any plan-gated fixes as `pending_enterprise`.
- **Enterprise need = SOFT** (1 inferred signal) → continue to self-service matching, recommend the best self-service plan, AND add a "you may also qualify for Enterprise" note + offer the Talk-to-Sales block.
- **Enterprise need = FALSE** → proceed normally below.
- **Already on Enterprise** → skip this gate; go straight to the ENTERPRISE branch (optimize/govern, no upsell).

Never quote or estimate a price for Enterprise (`data-integrity-rules.md` "Contact us" rule).

## Plan Decision Tree

> **MAU thresholds below are relative to `track_ceiling`** — the tenant's published MAU limit for its own track + plan, read from [pricing.md](pricing.md) (Free 25k; B2C Essentials 50k / Professional 30k; B2B Essentials 20k / Professional ~20k). "Approaching" means `current_mau ≥ 80% of track_ceiling`. Never compare against a flat 40k/50k number.

### Current Plan: FREE

IF use_case == "B2B" OR business_model == "B2B"
   AND (critical_gaps CONTAINS "Organizations" OR "Enterprise Connections" OR "Custom Domain")
   
   THEN: Recommend B2B Essentials
   REASON: Multi-tenant B2B needs Organizations, Enterprise Connections, Custom Domain
   FEATURES UNLOCK: Organizations, Enterprise Connections (3 included), Custom Domain, Pro MFA, RBAC, Log Streaming
   
ELSE IF use_case == "B2C" OR business_model == "B2C"
   AND (critical_gaps CONTAINS "Custom Domain" OR "Social Connections")
   
   THEN: Recommend B2C Essentials
   REASON: Consumer apps need Custom Domain, social auth, MFA
   FEATURES UNLOCK: Custom Domain, Pro MFA, Email Provider, Branding, Social Connections (unlimited)
   
ELSE IF ai_use_case == true AND integrations.length > 0 AND a4aa_fit_score ≥ 0.4
   AND autonomous_actions DETECTED (sending emails, financial transactions, modifying records)
   
   THEN: Recommend B2B Professional + A4AA add-on
   REASON: Token Vault + CIBA require Professional base or higher + A4AA add-on
   FEATURES UNLOCK: Token Vault, CIBA, M2M Access for Organizations, Enhanced MFA
   
ELSE IF ai_use_case == true AND integrations.length > 0 AND a4aa_fit_score ≥ 0.4
   AND autonomous_actions NOT detected (read-only or passive agents)
   
   THEN: Recommend B2B Essentials + A4AA add-on
   REASON: Token Vault sufficient for passive AI workflows
   FEATURES UNLOCK: Token Vault (basic), M2M Authentication
   
ELSE IF compliance_vertical DETECTED (fintech, healthcare, education, government)
   
   THEN: Recommend B2B Essentials
   REASON: Regulated verticals need Log Streaming, MFA enforcement, Breached Password Detection
   FEATURES UNLOCK: Log Streaming, MFA enforcement, Email Provider, Branding
   
ELSE IF no clear signals OR mixed use case
   
   THEN: Default to B2C Essentials
   REASON: Essentials is the entry point for production use
   FEATURES UNLOCK: Custom Domain, Pro MFA, Email Provider, Social Connections

---

### Current Plan: B2C ESSENTIALS

IF readiness_score > 80% AND current_mau < 80% of track_ceiling AND no_critical_gaps
   
   THEN: Recommend STAY on B2C Essentials
   STATUS: "Well-fitted for current use case"
   ACTION: No upgrade needed; monitor MAU growth
   
ELSE IF current_mau ≥ 80% of track_ceiling OR (monthly_growth > 20% AND months_until_essentials_limit < 6)
   
   THEN: Recommend upgrade to B2C Professional
   REASON: Approaching the Essentials MAU ceiling for this track (per pricing.md); need higher tier
   FEATURES UNLOCK: Enhanced Password Protection, Breached Password Detection, Security Center, Custom Database Connections
   
ELSE IF critical_gaps CONTAINS "Custom DB Connections" OR "Enhanced Password Protection" OR "5k+ M2M tokens/month"
   
   THEN: Recommend upgrade to B2C Professional
   REASON: These features require Professional plan
   FEATURES UNLOCK: Enhanced Password Protection, Breached Password Detection, Custom Database Connections, Security Center

---

### Current Plan: B2C PROFESSIONAL

IF readiness_score > 90% AND current_mau < 80% of track_ceiling
   
   THEN: Recommend STAY on B2C Professional
   STATUS: "Optimal configuration for use case"
   ACTION: Monitor for additional needs; no upgrade required
   
ELSE IF current_mau ≥ 80% of track_ceiling OR (monthly_growth > 20% AND months_until_professional_limit < 3)
   
   THEN: Recommend contact Enterprise sales
   REASON: Approaching the Professional MAU ceiling for this track (per pricing.md); custom contract needed
   STATUS: "You've outgrown standard plans"
   ACTION: "Contact Auth0 sales for Enterprise plan options"

---

### Current Plan: B2B ESSENTIALS

IF readiness_score > 80% AND current_mau < 80% of track_ceiling AND no_critical_gaps
   
   THEN: Recommend STAY on B2B Essentials
   STATUS: "Well-fitted for current use case"
   ACTION: No upgrade needed; monitor MAU growth
   
ELSE IF current_mau ≥ 80% of track_ceiling OR (monthly_growth > 20% AND months_until_essentials_limit < 6)
   
   THEN: Recommend upgrade to B2B Professional
   REASON: Approaching the Essentials MAU ceiling for this track (per pricing.md); need higher tier
   FEATURES UNLOCK: 5 Enterprise Connections (vs 3), Enhanced Password Protection, Breached Password Detection, Security Center, Custom Database Connections
   
ELSE IF critical_gaps CONTAINS "Custom DB Connections" OR "4+ Enterprise Connections" OR "M2M Access for Organizations" OR "5k+ M2M tokens/month"
   
   THEN: Recommend upgrade to B2B Professional
   REASON: These features require Professional plan
   FEATURES UNLOCK: 5 Enterprise Connections, M2M Access for Organizations, Enhanced Password Protection, Breached Password Detection, Security Center, Custom Database Connections
   
ELSE IF ai_use_case == true AND integrations.length > 0 AND a4aa_fit_score ≥ 0.4
   
   THEN: Recommend B2B Essentials + A4AA add-on
   REASON: Token Vault sufficient for AI workflows on Essentials base
   FEATURES UNLOCK: Token Vault, M2M token pool
   IF autonomous_actions DETECTED:
      THEN: Recommend upgrade to B2B Professional + A4AA instead
      REASON: CIBA requires Professional base
      FEATURES UNLOCK: CIBA, Token Vault, Enhanced MFA

---

### Current Plan: B2B PROFESSIONAL

IF readiness_score > 90% AND current_mau < 80% of track_ceiling
   
   THEN: Recommend STAY on B2B Professional
   STATUS: "Optimal configuration for use case"
   ACTION: Monitor for additional needs; no upgrade required
   
ELSE IF current_mau ≥ 80% of track_ceiling OR (monthly_growth > 20% AND months_until_professional_limit < 3)
   
   THEN: Recommend contact Enterprise sales
   REASON: Approaching the Professional MAU ceiling for this track (per pricing.md); custom contract needed
   STATUS: "You've outgrown standard plans"
   ACTION: "Contact Auth0 sales for Enterprise plan options"
   
ELSE IF ai_use_case == true AND integrations.length > 0 AND a4aa_fit_score ≥ 0.4
   
   THEN: Recommend B2B Professional + A4AA add-on
   REASON: Unlock Token Vault, CIBA, M2M token pool for AI agents
   FEATURES UNLOCK: Token Vault (unlimited), CIBA (all forms), Enhanced M2M token pool

---

### Current Plan: ENTERPRISE

**No upsell.** The customer already owns the top tier — recommendations focus on **optimization, governance, and adopting features they already own**: enforce/raise MFA assurance, enable Continuous Session Protection, Adaptive MFA, Bot Detection / Credential Guard (if licensed), tighten Tenant ACLs, route Prioritized Security Log Streams to their SIEM, adopt the FAPI profile where relevant, and govern Organizations/RBAC at scale. Frame each as "you already have this — here's how to get value from it," never as a purchase. Only the A4AA add-on may be *suggested* (it's a genuine add, not a tier change).

IF readiness_score > 90% AND custom_sla_active
   
   THEN: Recommend STAY on Enterprise
   STATUS: "Optimized for scale and compliance"
   ACTION: Continue with Auth0 support team; no action needed
   
ELSE IF ai_use_case == true AND integrations.length > 0 AND a4aa_fit_score ≥ 0.4
   
   THEN: Recommend add A4AA to existing Enterprise contract
   REASON: Unlock Token Vault, CIBA, M2M token pool for AI agents
   ACTION: "Contact your Auth0 account team to add A4AA to your contract"

---

## MAU Forecast Calculation

### Input Data
- `current_mau` (number) — current monthly active users
- `monthly_growth_rate` (percentage) — e.g., 15 means 15% per month

### Formula

MAU_at_month_N = current_mau × (1 + growth_rate) ^ N

### Example
Current MAU: 500
Monthly growth: 15% (0.15)

Month 1: ~575 · Month 12: ~2,675 · Month 28: ~25,033 ← reaches Free tier limit (25k)

Result: "At 15% monthly growth from 500 MAU you reach the Free tier's 25,000-MAU limit in ~28 months — MAU urgency is LOW; choose a plan on feature-fit, not capacity."

### Plan Tier Limits

> **`references/pricing.md` is the source of truth — never hardcode; these must match it. Compare MAU against the customer's own track (B2C vs B2B), not a flat number.**

- Free: **25,000 MAU** (B2C and B2B)
- B2C Essentials: up to 50,000 MAU · B2C Professional: up to 30,000 MAU
- B2B Essentials: up to 20,000 MAU · B2B Professional: up to ~20,000 MAU (beyond = Contact us)
- Enterprise: Custom (contact sales)

Note: the decision tree compares `current_mau` against `track_ceiling` — the tenant's published per-track MAU limit (read from pricing.md), not a flat number. "Approaching" means `current_mau ≥ 80% of track_ceiling`.

### Output Format

Forecast Timeline:
- Current MAU: 500
- Monthly growth rate: 15%
- Months until Free tier limit (25k): ~28 months
- MAU urgency: LOW (capacity is not the constraint at this growth)
- Recommended upgrade timing: based on feature-fit gaps, not the MAU clock

---

## Feature Unlock Matrix: Free → Each Plan

### Free → B2C Essentials

FEATURES THAT UNLOCK:
✓ Custom Domain (1 included)
✓ Pro MFA Factors (WebAuthn, Authenticator App)
✓ Email Workflow & Branding
✓ Customize Signup & Login
✓ Log Streaming (1 stream included)
✓ Social Connections (unlimited)
✓ 10 Organizations
✓ Email Provider (configurable)

FEATURES THAT REMAIN THE SAME:
= Core auth flow (Email/Password)
= Database connections
= Application registration

### Free → B2C Professional

FEATURES THAT UNLOCK:
✓ Custom Domain (1 included)
✓ Pro MFA Factors (WebAuthn, Authenticator App)
✓ Email Workflow & Branding
✓ Customize Signup & Login
✓ Log Streaming (1 stream included)
✓ Social Connections (unlimited)
✓ 10 Organizations
✓ Email Provider (configurable)
✓ Enhanced Password Protection
✓ Breached Password Detection
✓ Security Center
✓ Custom Database Connections
✓ M2M Tokens (5,000 included)

### Free → B2B Essentials

FEATURES THAT UNLOCK:
✓ Custom Domain (1 included)
✓ Auth0 Organizations (unlimited)
✓ Enterprise Connections (SAML/OIDC) — 3 included
✓ Pro MFA Factors (WebAuthn, Authenticator App)
✓ RBAC (Roles & Permissions)
✓ Log Streaming (1 stream included)
✓ Email Workflow & Branding
✓ Email Provider (configurable)
✓ Per-Organization Branding

FEATURES THAT REMAIN THE SAME:
= Core auth flow (Email/Password)
= Database connections
= Application registration

### Free → B2B Professional

FEATURES THAT UNLOCK:
✓ Custom Domain (1 included)
✓ Auth0 Organizations (unlimited)
✓ Enterprise Connections (SAML/OIDC) — 5 included
✓ Pro MFA Factors (WebAuthn, Authenticator App)
✓ Enterprise MFA Factors (included, not add-on)
✓ RBAC (Roles & Permissions)
✓ Log Streaming (1 stream included)
✓ Email Workflow & Branding
✓ Email Provider (configurable)
✓ Per-Organization Branding
✓ Enhanced Password Protection
✓ Breached Password Detection
✓ Security Center
✓ Custom Database Connections
✓ M2M Tokens (5,000 included)
✓ M2M Access for Organizations

---

## A4AA (Auth for AI Agents) Add-On Logic

### Detect A4AA Fit

IF ai_use_case == true 
   AND integrations.length > 0 
   AND integrations CONTAINS (Gmail, Slack, Salesforce, Stripe, HubSpot, GitHub, Jira, etc.)
   AND a4aa_fit_score ≥ 0.4
   
   THEN: A4AA is relevant
   
### Assess Tier Requirement

IF autonomous_actions DETECTED (sending emails, charging customers, modifying records, publishing)
   
   THEN: Requires B2B Professional + A4AA
   FEATURES: Token Vault, CIBA (async approval), Enhanced M2M token pool
   
ELSE IF read-only OR passive agent flows
   
   THEN: B2B Essentials + A4AA is sufficient
   FEATURES: Token Vault (basic), M2M token pool

### A4AA Pricing (from pricing.md)

A4AA adds 50% to base price (rounded up)

Example:
- B2B Essentials at 1k MAU: $300/month base
- A4AA add-on: 50% × $300 = $150/month
- Total: $450/month

---

## Output: Plan Recommendation JSON

{
  "current_plan": "Free",
  "recommended_plan": "B2B Essentials",
  "a4aa_recommended": true,
  "mau_forecast": {
    "current_mau": 500,
    "monthly_growth_rate": 0.15,
    "months_until_free_limit": 28,
    "forecast_note": "At 15% monthly growth from 500 MAU, the Free tier's 25,000-MAU limit is ~28 months away — low MAU urgency"
  },
  "feature_unlocks": [
    {
      "feature": "Custom Domain",
      "severity": "CRITICAL",
      "reason": "Enterprise customers need branded auth endpoints"
    },
    {
      "feature": "Organizations",
      "severity": "CRITICAL",
      "reason": "Multi-tenant B2B requires per-customer isolation"
    },
    {
      "feature": "Enterprise Connections (3 included)",
      "severity": "CRITICAL",
      "reason": "Enterprise customers need SSO with their corporate IdP"
    },
    {
      "feature": "MFA Enforcement",
      "severity": "HIGH",
      "reason": "Enterprises require MFA in procurement"
    },
    {
      "feature": "Log Streaming",
      "severity": "HIGH",
      "reason": "Compliance + audit trail for enterprise customers"
    }
  ],
  "a4aa_features": [
    {
      "feature": "Token Vault",
      "reason": "Securely store + rotate OAuth tokens for AI agents"
    },
    {
      "feature": "CIBA",
      "reason": "Async user approval for high-stakes agent actions"
    }
  ],
  "estimated_cost": "$300/month B2B Essentials @ 1k MAU + $150/month A4AA = $450/month"
}