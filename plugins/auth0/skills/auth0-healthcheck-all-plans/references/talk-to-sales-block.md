# Talk-to-Sales Block

Emitted only when [enterprise-need-detection.md](enterprise-need-detection.md) returns `TRUE` or `SOFT`. Appears in **both** the in-chat summary and the report (Part B enterprise/soft variant).

Goal: make the sales reach-out one copy-paste. The skill already knows everything sales would ask, so it pre-fills the brief. The skill **cannot** submit a form or create a CRM lead on the user's behalf — it provides a ready-to-send brief + a contact link.

## Template

```
**Talk to Sales — prefilled brief**

- Company:             {{customer_name}} ({{company_domain}})
- Use case:            {{detected_use_case}} — {{product_summary_short}}
- Current Auth0 plan:  {{current_plan}}
- MAU + growth:        {{current_mau}} MAU, {{monthly_growth_rate}}%/mo ({{mau_forecast_note}})
- Enterprise features needed: {{enterprise_features_list}}
- Top capability gaps: {{top_gaps_list}}
- Why now (triggers):  {{enterprise_triggers_list}}

Contact Auth0 sales: https://auth0.com/contact-us
{{custom_ae_link_slot}}
```

## Token sources

| Token | Source |
|---|---|
| `{{customer_name}}`, `{{company_domain}}` | enrichment / Phase 0 |
| `{{detected_use_case}}`, `{{product_summary_short}}` | Phase 2 + enrichment |
| `{{current_plan}}` | Phase 0 normalized facts |
| `{{current_mau}}`, `{{monthly_growth_rate}}`, `{{mau_forecast_note}}` | Phase 4 MAU forecast |
| `{{enterprise_features_list}}` | enterprise-need-detection output `enterprise_features_needed` |
| `{{top_gaps_list}}` | Phase 3B CRITICAL/HIGH gaps |
| `{{enterprise_triggers_list}}` | enterprise-need-detection output `triggers` |
| `{{custom_ae_link_slot}}` | `state/operator.json.ae_link` if set; **omit the line entirely if unset** |

## Variants

- **TRUE:** lead with "Auth0 Enterprise is the right fit — here's a brief to start the conversation." No price anywhere.
- **SOFT:** lead with "You may also qualify for Enterprise; here's a brief if you'd like to explore it" — shown *in addition* to the recommended self-service plan + cost.

## Rules

- **Never include a price or estimate** for Enterprise.
- Keep the brief factual and short — it's for a human to send, not marketing copy.
- If enrichment confidence is low, add: "Some company details are estimated — correct them before sending."
