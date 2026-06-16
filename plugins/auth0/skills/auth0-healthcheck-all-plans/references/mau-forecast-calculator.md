# MAU Forecast Calculator

## Input Data Required

- current_mau (number) — current monthly active users
- monthly_growth_rate (percentage) — e.g., 15 means 15% per month
- current_plan (string) — Free, B2C Essentials, B2C Professional, B2B Essentials, B2B Professional, Enterprise

## Plan Tier Limits

> **`references/pricing.md` is the source of truth for these limits — never hardcode; the values below must match it. Always compare MAU against the customer's own track (B2C vs B2B).**

- Free: **25,000 MAU** (identical for B2C and B2B)
- B2C Essentials: up to 50,000 MAU
- B2C Professional: up to 30,000 MAU (40k+ not available)
- B2B Essentials: up to 20,000 MAU (30k+ = Contact us)
- B2B Professional: up to ~20,000 MAU (beyond = Contact us)
- Enterprise: Custom (contact sales)

## MAU Forecast Formula

MAU_at_month_N = current_mau × (1 + growth_rate) ^ N

Where:
- current_mau = starting MAU
- growth_rate = monthly growth as decimal (e.g., 0.15 for 15%)
- N = month number

## Example Calculation

Input:
- Current MAU: 500
- Monthly growth rate: 15% (0.15)

Forecast:
Month 1: 500
Month 6: ~1,005
Month 12: ~2,456
Month 24: ~12,000
Month 28: ~25,000 ← reaches the Free tier limit (25k)

Result:
- Months until Free tier limit (25k): ~28 months (~2.3 years)
- MAU urgency: LOW — ample runway on Free at this growth
- Upgrade timing: driven by feature-fit gaps (B2B Organizations, SSO, MFA, etc.), NOT by capacity. Recommend on use-case need, not the MAU clock.

## Output Format

Forecast Timeline:
- Current MAU: 500
- Monthly growth rate: 15%
- Months until Free tier limit (25k): ~28 months
- MAU urgency: LOW (capacity is not the constraint at this growth)
- Recommended upgrade timing: based on feature-fit gaps, not the MAU clock (see plan-matching-logic.md)

---

## Interactive Fallback Logic

If current MAU is NOT available from tenant telemetry:

PROMPT USER:
"What's your estimated monthly growth rate?
Examples:
- 5% (conservative, slow growth)
- 15% (typical SaaS)
- 30% (aggressive growth)
- 50%+ (hypergrowth)
Enter as percentage (e.g., 15 for 15%)"

IF user provides growth_rate:
   THEN: Use user_provided_growth_rate
   
ELSE IF user skips or says "I don't know":
   THEN: DEFAULT to 15% as conservative SaaS estimate
   OUTPUT NOTE: "Forecast assumes 15% monthly growth (typical for SaaS). Adjust based on your actual growth."

---

## Forecast Interpretation Guide

### Months Until Limit < 3 months
**Urgency: CRITICAL**
- User will exceed current tier limit very soon
- Recommend immediate upgrade
- Provide specific plan recommendation
- Include action items for this month

### Months Until Limit 3–6 months
**Urgency: HIGH**
- User will exceed limit within 6 months
- Recommend upgrade planning now
- Include implementation timeline

### Months Until Limit 6–12 months
**Urgency: MODERATE**
- User has time to plan
- Recommend upgrade before approaching limit
- Include feature unlock benefits

### Months Until Limit > 12 months
**Urgency: LOW**
- User has ample runway
- Recommend for future roadmap
- Focus on use-case fit over timing

---

## Edge Cases

### Edge Case 1: Current MAU Already Exceeds Plan Limit

IF current_mau > plan_limit:
   
   THEN: 
   Status: "OVERAGES DETECTED"
   Message: "Your current MAU ({current_mau}) exceeds your plan limit ({plan_limit}). You may be incurring overage charges or hitting hard limits."
   Action: "Contact Auth0 sales immediately to upgrade or clarify billing."

### Edge Case 2: Current MAU = 0 or Not Available

IF current_mau == 0 OR current_mau == null:
   
   THEN:
   Prompt: "What's your current monthly active user count? (Leave blank if unknown)"
   IF user provides number:
      THEN: Use provided number
   ELSE IF user leaves blank:
      THEN: Use 100 as default estimate
      NOTE: "Forecast assumes 100 MAU as starting point. Update with actual data for accuracy."

### Edge Case 3: Growth Rate = 0% or Negative

IF growth_rate <= 0:
   
   THEN:
   Status: "FLAT OR DECLINING"
   Message: "No growth detected or negative growth. No upgrade urgency from MAU forecast perspective."
   ACTION: "Consider upgrade based on use-case fit (B2B, compliance, AI agents) rather than growth."

### Edge Case 4: User on Enterprise

IF current_plan == "Enterprise":
   
   THEN:
   Status: "CUSTOM PLAN"
   Message: "You're on a custom Enterprise plan with no hard MAU limit. Forecast does not apply."
   ACTION: "Contact your Auth0 account team for scaling guidance."

---

## Output JSON

{
  "current_mau": 500,
  "monthly_growth_rate": 0.15,
  "forecast": {
    "months_until_free_limit": 28,
    "forecast_note": "At 15% monthly growth from 500 MAU, you reach the Free tier's 25,000-MAU limit in ~28 months"
  },
  "urgency": "LOW",
  "recommended_action": "No MAU-driven urgency; recommend a plan based on use-case feature gaps, not capacity",
  "timeline": {
    "month_1_mau": 575,
    "month_6_mau": 1005,
    "month_12_mau": 2456,
    "month_24_mau": 12029,
    "month_28_mau": 25000
  }
}