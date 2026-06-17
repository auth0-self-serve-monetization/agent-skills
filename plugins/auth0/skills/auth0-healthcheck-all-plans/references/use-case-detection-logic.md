# Use-Case Detection Logic

## Decision Tree

### Step 1: Check for Regulated Verticals

IF company_description CONTAINS (fintech | banking | payments | healthcare | medical | government | HIPAA | SOC2 | PCI)
   OR compliance_vertical IN (fintech, healthcare, education, government, legal)
   
   THEN: Classified as REGULATED VERTICAL
   
   SURFACE FEATURES:
   • Log Streaming (audit trails for compliance)
   • Breached Password Detection (security baseline)
   • Session Management (track & revoke sessions)
   • MFA enforcement (required for regulated)
   • Custom Domain (credibility + brand trust)
   • Organizations (multi-tenant isolation if B2B)
   • Enterprise Connections (if enterprise customers exist)
   
   ADDITIONAL FOR FINTECH:
   • Anomaly Detection
   • Risk-based authentication
   • Token rotation policies
   
   ADDITIONAL FOR HEALTHCARE:
   • HIPAA-eligible log retention
   • Encrypted log storage
   • Audit trail immutability

### Step 2: Check Business Model

IF business_model == "B2B" OR customer_segments CONTAINS (enterprise, SMB)
   
   THEN: Classified as B2B
   
   SURFACE FEATURES:
   • Auth0 Organizations (multi-tenant management)
   • Enterprise Connections (SAML/OIDC — usually 3–5 included)
   • Custom Domain (REQUIRED for credibility)
   • Roles & RBAC (permission management)
   • MFA enforcement (enterprises demand it)
   • Log Streaming (compliance + audit)
   • Email provider + branded templates
   
   GAP SEVERITY:
   - Custom Domain: CRITICAL (missing = credibility killer)
   - Organizations: CRITICAL (missing = can't isolate customers)
   - Enterprise Connections: HIGH (missing = can't SSO with corporate IdPs)
   - MFA: HIGH (enterprises require it in procurement)
   - Log Streaming: MODERATE (compliance + observability)

ELSE IF business_model == "B2C" OR customer_segments CONTAINS (consumer)
   
   THEN: Classified as B2C
   
   SURFACE FEATURES:
   • Social Connections (Google, GitHub, Facebook, LinkedIn, etc.)
   • Custom Domain (branded login experience)
   • Email/Password connection with good UX
   • MFA optional (depends on app sensitivity)
   • Passwordless (phone/email/biometric) — nice-to-have
   • Bot detection (prevent automated attacks)
   
   GAP SEVERITY:
   - Social Connections: MODERATE (multi-auth improves conversion)
   - Custom Domain: MODERATE (brand trust)
   - Email/Password UX: LOW (default usually fine)
   - MFA: LOW (unless app handles sensitive data)

ELSE
   
   THEN: Classified as MIXED (B2B + B2C)
   
   SURFACE FEATURES:
   • Both B2B AND B2C feature sets
   • Organizations (for B2B tenants)
   • Social Connections (for B2C users)
   • Custom Domain (required for both)

### Step 3: Check for AI Use Case

IF ai_use_case == true 
   AND integrations.length > 0
   AND integrations CONTAINS (Gmail, Slack, Salesforce, GitHub, Stripe, HubSpot, Jira, etc.)
   
   THEN: Classified as AI-NATIVE OR AI-DIFFERENTIATED
   
   SURFACE FEATURES:
   • Token Vault (securely store + refresh OAuth tokens for agent integrations)
     - List specific APIs: "e.g., Gmail, Slack, Salesforce"
   
   • CIBA (Client Initiated Backchannel Authentication — async approval for high-stakes actions)
     - IF autonomous_actions CONTAINS (sending emails, updating records, publishing, financial transactions)
     - THEN: CIBA is critical
   
   • M2M Authentication (machine-to-machine for agent-to-API flows)
   
   • Custom Claims (pass agent context in tokens)
   
   GAP SEVERITY:
   - Token Vault: CRITICAL (missing = security risk + manual token management)
   - CIBA: HIGH (if autonomous actions detected; missing = no human approval trail)
   - M2M Auth: MODERATE (needed for agent orchestration)
   - Custom Claims: LOW (optimization)

ELSE IF ai_use_case == true 
   AND integrations.length == 0
   
   THEN: Classified as AI-ADJACENT (LLM integration but no OAuth/external APIs)
   
   SURFACE FEATURES:
   • M2M Authentication (for LLM API calls)
   • Custom Claims (context passing)
   
   GAP SEVERITY:
   - M2M Auth: MODERATE
   - Token Vault: NOT RELEVANT
   - CIBA: NOT RELEVANT

## Feature Gap Severity Classification

### CRITICAL Gaps
- Custom Domain (missing = brand credibility killer)
- Organizations (B2B missing = can't isolate customers)
- Enterprise Connections (B2B missing = can't SSO with corporate IdPs)
- Token Vault (AI missing = OAuth token security risk)
- Log Streaming (regulated vertical missing = compliance failure)

### HIGH Gaps
- MFA enforcement (enterprises require in procurement)
- CIBA (autonomous actions missing = no approval trail)
- Roles/RBAC (B2B without roles = no permission granularity)

### MODERATE Gaps
- Email provider + branding (B2B without = unprofessional)
- Social Connections (B2C without = lower conversion)
- M2M Auth (AI without = manual agent orchestration)
- Anomaly Detection (regulated without = security blind spot)

### LOW Gaps
- Session Management tweaks
- Passwordless (nice-to-have, not blocking)
- Bot Detection (unless abuse detected)

## Output: Use-Case Recommendation

For each detected use case, return:

{
  "detected_use_case": "B2B SaaS + AI Agents",
  "verticals": ["fintech"],
  "business_model": "B2B",
  "ai_use_case": true,
  "ai_integrations": ["Salesforce", "Stripe", "Gmail"],
  "required_features": [
    "Custom Domain",
    "Organizations",
    "Enterprise Connections",
    "MFA enforcement",
    "Log Streaming",
    "Token Vault",
    "CIBA"
  ],
  "configured_features": [
    "Email/Password auth",
    "Applications registered"
  ],
  "missing_features": [
    {
      "feature": "Custom Domain",
      "severity": "CRITICAL",
      "reason": "Enterprise customers see generic Auth0 domain at login"
    },
    {
      "feature": "Organizations",
      "severity": "CRITICAL",
      "reason": "Can't isolate each fintech customer's data + branding"
    },
    {
      "feature": "Token Vault",
      "severity": "CRITICAL",
      "reason": "AI agents need secure OAuth token storage for Salesforce + Stripe integrations"
    }
  ],
  "readiness_score": 0.35,
  "readiness_level": "Not Ready"
}