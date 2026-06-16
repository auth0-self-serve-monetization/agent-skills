# Feature Recommendations by Use Case

## B2B SaaS (Enterprise Customers)

| Feature | Why It Matters | CheckMate Signal | Action |
|---------|----------------|------------------|--------|
| Custom Domain | Enterprises see your-domain.com/auth instead of company.us.auth0.com — kills credibility | custom_domains_present == false | CRITICAL: Configure immediately |
| Organizations | Isolate each customer's data, branding, users, and roles per tenant | organizations_configured == false | CRITICAL: Required for multi-tenant |
| Enterprise Connections (SAML/OIDC) | Let customers SSO with their corporate IdP (Azure AD, Okta, etc.) | enterprise_connections_count == 0 | CRITICAL: Table-stakes for enterprise deals |
| MFA (WebAuthn/Authenticator App) | Enterprises require it in procurement; reduces account takeover risk | mfa_configured == false | HIGH: Enable on all apps |
| Roles & RBAC | Manage permissions per role (admin, editor, viewer, etc.) | roles_configured == false | HIGH: Required for teams |
| Log Streaming | Stream logs to SIEM for compliance, audit trails, incident response | log_streams_present == false | MODERATE: Required for SOC2/ISO27001 |
| Email Provider + Branding | Send branded password reset/invitation emails | email_provider_configured == false | MODERATE: Improves UX |
| Session Management | Track, rotate, and revoke sessions; enforce timeout policies | session_management_configured == false | LOW: Nice-to-have unless regulated |

## B2C App (Consumer Users)

| Feature | Why It Matters | CheckMate Signal | Action |
|---------|----------------|------------------|--------|
| Social Connections | Let users sign up with Google, GitHub, Facebook, LinkedIn | social_connections_count < 2 | HIGH: Increases signup conversion |
| Custom Domain | Keep users on-brand during login (e.g., auth.myapp.com) | custom_domains_present == false | MODERATE: Important for brand trust |
| Email/Password UX | Passwordless (email link, SMS OTP, or WebAuthn) improves experience | passwordless_configured == false | MODERATE: Reduces friction |
| MFA Optional | WebAuthn or Authenticator App for security-conscious users | mfa_configured == false | LOW: Optional unless handling sensitive data |
| Bot Detection | Prevent automated signup/login attacks | bot_detection_configured == false | LOW: Only if abuse detected |
| Brute Force Protection | Limit failed login attempts | brute_force_protection_enabled == false | MODERATE: Security baseline |

## AI-Native Platform (Autonomous Agents)

| Feature | Why It Matters | CheckMate Signal | Action |
|---------|----------------|------------------|--------|
| Token Vault | Securely store, refresh, and rotate OAuth tokens for agent integrations (Gmail, Slack, Salesforce, etc.) without secrets in code | token_vault_enabled == false | CRITICAL: Non-negotiable for OAuth |
| CIBA (Client Initiated Backchannel Auth) | Enable async user approval for high-stakes agent actions (sending emails, financial transactions, modifying data) | ciba_configured == false | CRITICAL if autonomous_actions detected |
| M2M Authentication | Agents authenticate to APIs without user context; manage with client credentials | m2m_apps_count == 0 | HIGH: Required for agent orchestration |
| Custom Claims | Pass agent context, user role, or action type in JWT claims for downstream APIs | custom_claims_configured == false | MODERATE: Optimization for agent logic |
| Log Streaming | Audit trail for all agent actions (who approved, what action, when) | log_streams_present == false | MODERATE: Compliance + debugging |

## Fintech / Regulated Verticals

| Feature | Why It Matters | CheckMate Signal | Action |
|---------|----------------|------------------|--------|
| Log Streaming | Required for SOX/HIPAA/PCI audit trails; stream to your SIEM | log_streams_present == false | CRITICAL: Compliance blocker |
| Custom Domain | Enterprises + regulators expect branded auth endpoints | custom_domains_present == false | CRITICAL: Credibility + compliance |
| Breached Password Detection | Automatically detect compromised credentials from data breaches | breached_password_detection_enabled == false | HIGH: Security baseline for fintech |
| MFA Enforcement | Required by compliance frameworks; enforce across all user flows | mfa_configured == false | CRITICAL: Non-negotiable |
| Anomaly Detection | Flag suspicious login attempts (unusual location, device, time) | anomaly_detection_enabled == false | MODERATE: Fraud prevention |
| Session Management | Enforce session timeout, rotation, and revocation policies | session_policies_configured == false | MODERATE: Compliance requirement |
| Organizations (if multi-tenant B2B fintech) | Isolate customer data per tenant; required for fintech multi-tenancy | organizations_configured == false | HIGH: Regulatory requirement |

## Feature Priority Matrix

PRIORITY   | VERTICAL              | FEATURE
-----------|----------------------|-----------------------
CRITICAL   | All                   | Custom Domain
CRITICAL   | B2B                   | Organizations
CRITICAL   | B2B                   | Enterprise Connections
CRITICAL   | AI                    | Token Vault
CRITICAL   | Regulated             | Log Streaming
CRITICAL   | Regulated             | MFA enforcement
HIGH       | B2B                   | RBAC
HIGH       | B2B                   | MFA enforcement
HIGH       | AI                    | CIBA
HIGH       | Regulated             | Breached Password Detection
MODERATE   | B2B                   | Email provider + branding
MODERATE   | B2C                   | Social Connections
MODERATE   | AI                    | M2M Authentication
MODERATE   | Regulated             | Anomaly Detection
LOW        | B2C                   | Passwordless
LOW        | All                   | Session Management tweaks