# Data Integrity Rules for Plan Fit Diagnostic

## Input Validation

### CheckMate Data

IF CheckMate report is missing or empty:
   THEN: Proceed with enrichment data only
   STATUS: "Security audit data unavailable; proceeding with use-case analysis"
   ACTION: Warn user that security findings won't be included in recommendations

IF CheckMate report contains zero findings (all passing):
   THEN: Classify as "Security Posture: Excellent"
   CONFIDENCE: High (no gaps detected)
   ACTION: Focus recommendations on use-case fit + feature optimization

### Enrichment Data

IF business_model is null or unknown:
   THEN: Ask user directly: "Is your use case B2B, B2C, or Mixed?"
   FALLBACK: Infer from domain name + company description
   CONFIDENCE: Low (0.3–0.5)

IF current_mau is null OR current_mau == 0:
   THEN: Ask user: "What's your current monthly active users?"
   FALLBACK: Use 100 as placeholder estimate
   OUTPUT NOTE: "Forecast assumes 100 MAU as baseline. Update with actual data for accuracy."

IF monthly_growth_rate is null OR monthly_growth_rate == 0:
   THEN: Ask user: "What's your estimated monthly growth rate? (e.g., 10%, 15%, 25%)"
   FALLBACK: Use 15% as conservative SaaS estimate
   OUTPUT NOTE: "Forecast assumes 15% monthly growth (typical for SaaS). Adjust based on your actual growth."

### Current Plan

IF current_plan not detected from tenant:
   THEN: Ask user: "What Auth0 plan are you currently on? (Free, B2C Essentials, B2B Essentials, Professional, Enterprise)"
   ACTION: Default to "Free" if user unsure

IF current_plan == "Enterprise":
   THEN: Skip standard plan recommendation logic
   ACTION: Recommend contacting Auth0 account team for custom assessment

---

## MAU Forecast Validation

### Bounds Checking

IF current_mau > 500,000:
   THEN: WARN "Unusually high MAU. Confirm accuracy."
   ACTION: Proceed with forecast, but flag for verification

IF monthly_growth_rate > 100% (100% per month):
   THEN: WARN "Hypergrowth (>100% monthly) detected. Very short runway."
   ACTION: Proceed, but emphasize urgency in output

IF monthly_growth_rate < 0% (negative growth):
   THEN: STATUS "Declining user base"
   ACTION: Focus recommendations on use-case fit, not growth-driven urgency

### Forecast Calculation Validation

FORMULA: MAU_at_month_N = current_mau × (1 + growth_rate) ^ N

VERIFY each calculation:
   - month_1_mau ≈ current_mau × (1 + growth_rate)
   - month_6_mau should be > current_mau (if positive growth)
   - months_until_limit should be positive integer
   - months_until_limit > 0 before exceeding plan limit

IF calculation fails (NaN, negative months, etc.):
   THEN: Return null for forecast
   ACTION: Prompt user to re-enter MAU + growth rate

---

## Plan Recommendation Validation

### Decision Tree Integrity

IF use_case == "Unknown":
   THEN: DEFAULT to "B2C Essentials"
   REASON: Conservative fallback; can upgrade later
   CONFIDENCE: Low (0.3)

IF use_case == "B2B" BUT Organizations NOT detected AND business_model indicates "multi-tenant":
   THEN: OVERRIDE to "B2B Essentials" (minimum for B2B)
   REASON: Organizations is critical for B2B
   SEVERITY: CRITICAL gap

IF use_case == "AI-Native" OR "AI-Differentiated" AND integrations.length > 0:
   THEN: Surface A4AA add-on as CRITICAL
   REASON: Token Vault non-negotiable for OAuth integrations
   CONFIDENCE: High (0.85+)

IF ai_use_case == true AND autonomous_actions detected AND current_plan < "Professional":
   THEN: Recommend Professional (not Essentials)
   REASON: CIBA requires Professional base
   OVERRIDE: Do not recommend Essentials + A4AA if autonomous actions detected

### Feature Gap Cross-Reference

CRITICAL gaps (must unlock plan feature):
   - Custom Domain → Essentials or above
   - Organizations → B2B Essentials or above
   - Enterprise Connections (3+) → B2B Professional or above
   - Token Vault → A4AA add-on (any base plan)
   - CIBA → A4AA + Professional base or above
   - Log Streaming → Essentials or above

IF current_plan already has required feature:
   THEN: Recommend STAY on current plan (unless other gaps exist)
   ACTION: Focus on feature optimization, not upgrade

IF current_plan DOES NOT have required feature:
   THEN: Recommend minimum plan that unlocks it

### A4AA Fit Score Validation

A4AA Fit Score = sum of:
   + 0.25 if ai_use_case == "AI-Native"
   + 0.20 if ai_use_case == "AI-Differentiated"
   + 0.15 if integrations.length >= 3 (Gmail, Slack, Salesforce, etc.)
   + 0.15 if autonomous_actions detected (sending emails, charging customers, etc.)
   + 0.10 if approval_workflow required (CIBA)
   + 0.05 if custom_claims needed (agent context in tokens)
   + 0.10 if m2m_apps_count > 0 (agent orchestration)

VALIDATE:
   IF a4aa_fit_score >= 0.40 → Recommend A4AA as CRITICAL
   IF a4aa_fit_score 0.20–0.39 → Recommend A4AA as HIGH priority
   IF a4aa_fit_score < 0.20 → A4AA is optional / not recommended

---

## Pricing Data Consistency

### Reference pricing.md Authoritative Tables

REQUIRED BEFORE OUTPUT:
   1. Lock B2C base pricing table (monthly + yearly)
   2. Lock B2B base pricing table (monthly + yearly)
   3. Lock A4AA add-on pricing (monthly + yearly)
   4. Lock M2M token add-on pricing
   5. Lock Enterprise SSO connection add-on pricing ($100/month per additional)
   6. Lock Enterprise MFA add-on pricing ($100/month for Essentials)

IF pricing data is stale OR "Contact us" appears in tier:
   THEN: Surface as "Contact Auth0 sales for custom quote"
   ACTION: Do NOT calculate or estimate unlisted prices

### Cost Calculation Rules

Total Monthly Cost = Base Price + sum(add-ons)

IF add-on price is null or "Contact us":
   THEN: Output "Base Price + [Add-on name] (contact sales)"
   ACTION: Do not assume a number

IF A4AA recommended:
   THEN: Cost = Base Price + (Base Price × 0.50 rounded up)
   VERIFY: Match against pricing.md A4AA table for accuracy

IF M2M tokens > 5,000 (on Professional):
   THEN: Calculate add-on cost from M2M token pricing table
   VERIFY: Match against pricing.md M2M table

---

## Output Validation

### 4-Layer Structure Completeness

Before generating output, VERIFY all 4 layers are present:

**Layer 1: What I Did**
   - ✓ Technical summary (1–2 sentences)
   - ✓ References current plan + use case + key findings

**Layer 2: What This Means For Your App**
   - ✓ Business-focused language (non-technical)
   - ✓ Names specific company products (not generic terms)
   - ✓ Explains business impact (deal blocking, urgency, etc.)
   - ✓ References MAU forecast (if applicable)

**Layer 3: Technical Details**
   - ✓ Current plan name + MAU limit
   - ✓ MAU forecast (current + growth rate + months until limit)
   - ✓ Recommended plan name + MAU limit
   - ✓ Feature unlocks (5–8 features with justification)
   - ✓ Cost context (feature-focused only, no business assumptions)

**Layer 4: What's Next**
   - ✓ Clear action items or copy-paste prompts
   - ✓ References specific plan features
   - ✓ If Enterprise: "Contact sales" path
   - ✓ If user already on a plan: upgrade path or confirmation of fit

### Language Consistency

MUST USE:
   - Plan names from pricing.md exactly (B2C Essentials, B2B Essentials, Professional, Enterprise)
   - Feature names from pricing.md exactly (Custom Domain, Organizations, Enterprise Connections, etc.)
   - Company product names (not generic "your app" or "platform")
   - Factual language only (no business assumptions like "costs about one deal")

MUST NOT USE:
   - Generic terms ("more features", "better security")
   - Pricing assumptions without data
   - Jargon without explanation
   - Off-brand terminology

---

## Confidence Scoring

### Set Confidence Score (0–1) Based On:

0.9–1.0: All data available + verified (MAU, growth, use case, current plan confirmed)
0.7–0.9: Most data available + minor gaps filled by user input
0.5–0.7: Use case clear, but MAU/growth estimated with user fallback
0.3–0.5: Use case inferred; MAU/growth using defaults; plan detected
0.0–0.3: Minimal data; mostly defaults used; high uncertainty

OUTPUT confidence_score in final recommendation:
   "Recommendation Confidence: 0.85 (most data available; growth rate user-provided)"

---

## Error Handling

### If Data Missing / Unusable

IF critical data missing (current_plan, use_case):
   THEN: HALT and ask user directly
   PROMPT: "To provide a tailored recommendation, I need:
      1. Your current Auth0 plan (Free, B2C Essentials, B2B Essentials, Professional, Enterprise)
      2. Your use case (B2B SaaS, B2C App, AI-Native, Fintech, Other)
      3. Your current MAU (or monthly growth rate)"

IF enrichment failed (company data unavailable):
   THEN: Proceed with tech-only assessment (plan fit based on CheckMate findings alone)
   OUTPUT NOTE: "Company intelligence unavailable; recommendation based on technical posture alone."

IF forecast calculation fails:
   THEN: Output null forecast + explain
   ACTION: Ask user to re-enter MAU and growth rate

### Output Fallbacks

IF no errors, output complete 4-layer recommendation.

IF some data missing, output partial recommendation:
   - Include all available layers
   - Flag missing data in each layer
   - Recommend user to provide missing data for more tailored advice

IF all data missing, output generic recommendation:
   - "To get a personalized plan recommendation, please provide your current plan and use case."
   - Include 5–7 generic decision trees (Free→Essentials, Free→Professional, etc.) for reference