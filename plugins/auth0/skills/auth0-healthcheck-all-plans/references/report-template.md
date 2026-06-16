<!--
  Locked markdown report template for auth0-healthcheck-all-plans.
  Phase 5 substitutes every {{placeholder}} with the same data feeding report-template.html.
  Two sections: Part A Security & Config Hygiene (universal) + Part B Capability Fit (tier-aware).
  Omit any conditional block the data does not support. No {{placeholder}} may survive to output.
-->

# Auth0 Tenant Health Check — {{customer_name}}
{{prepared_by_line}}  <!-- "Prepared by {{reviewer_org}}" if reviewer_org non-empty; else omit this line -->

**{{plan_tag}}** · {{customer_name}} · {{tenant_domain}} · {{review_date_long}}

|  |  |
|---|---|
| **Security & Config Hygiene** | **{{hygiene_score}}** — {{hygiene_band}} (confidence {{hygiene_confidence}}) |
| **Capability Fit** | **{{fit_score}}** — {{fit_band}} (confidence {{fit_confidence}}) |

| Field | Value |
|---|---|
| Prepared by | {{reviewer_name}} |
| Date of Review | {{review_date_long}} |
| Customer Account | {{customer_name}} ({{tenant_domain}}) |
| Current Auth0 Plan | {{current_plan}} |
| Use Case Diagnosis | {{business_model}} — {{product_summary_short}} |

---

## Part A — Security & Config Hygiene

*Applies on every plan, Free → Enterprise. Score: {{hygiene_score}} — {{hygiene_band}}.*
<!-- If no CheckMate scan: replace score with "Not scored — run a CheckMate audit for a security score" and keep the disclaimer in {{confidence_note}}. -->

### Account Health

| Metric | Status |
|---|---|
| Applications Registered | {{apps_count}} |
| Login Activity | {{login_activity}} |
| Security Findings | {{findings_critical_count}} Critical · {{findings_warning_count}} Warning · {{findings_info_count}} Info · {{findings_passing_count}} Passing |

{{gaps_markdown}}  <!-- per supported gap: "**{{gap_name}}** — {{gap_body}}" (each names a real product/app/segment) -->

### Configuration Posture at a Glance

| Severity | Count | Key Examples |
|---|---|---|
| Critical | {{findings_critical_count}} | {{findings_critical_examples}} |
| Warning | {{findings_warning_count}} | {{findings_warning_examples}} |
| Info | {{findings_info_count}} | {{findings_info_examples}} |
| Passing | {{findings_passing_count}} | {{findings_passing_examples}} |

### Immediate Actions — Available Today on {{current_plan}}

{{immediate_actions_markdown}}  <!-- numbered list; each names affected apps/connections explicitly -->

---

## Part B — Capability Fit

*For your use case ({{detected_use_case}}) on {{current_plan}}. Score: {{fit_score}} — {{fit_band}}.*

### Feature-Gap Matrix

| Feature | Status | Plan Home | Why it matters |
|---|---|---|---|
{{gap_matrix_rows}}  <!-- per feature: "| {{feature}} | ✅/❌/⚠️ | Available now / Unlocks on {{plan}} / Enterprise-only | {{why_personalized}} |" -->

### Strategic Opportunities

| Feature Area | The Challenge | The Solution | Strategic Impact |
|---|---|---|---|
{{opportunities_rows}}  <!-- every Challenge cell names a specific product/segment from enrichment; omit a row that can't be made specific -->

### Plan Guidance

<!-- Render exactly ONE variant into {{plan_guidance_block}}:

SELF-SERVICE:
**Recommended Plan: {{recommended_plan}} — {{exact_cost}}{{a4aa_addon_suffix}}**
{{plan_rationale_personalized}}
**After Upgrading**
{{after_upgrading_markdown}}

ENTERPRISE (no price):
**Enterprise — contact sales**
{{enterprise_rationale}} Triggered by: {{enterprise_triggers_list}}.
{{talk_to_sales_brief}}
Contact Auth0 sales: https://auth0.com/contact-us {{custom_ae_link_md}}

ALREADY-ENTERPRISE (optimize; no upsell):
**You're on Enterprise — optimize what you already own**
{{optimization_actions_markdown}}
-->
{{plan_guidance_block}}

### Key Documentation

{{docs_markdown}}  <!-- 2-column list of doc links matched to the opportunities above -->

---

*{{confidence_note}}*
*Confidential · Auth0 tenant health check for {{customer_name}} · {{review_month_year}}*
