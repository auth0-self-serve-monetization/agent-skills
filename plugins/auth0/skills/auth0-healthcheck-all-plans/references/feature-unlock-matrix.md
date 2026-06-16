# Feature Unlock Matrix

> **`references/pricing.md` is the source of truth for feature availability — this file must not contradict it.**
> **The Free tier already includes: 1 Custom Domain, 5 Organizations, 1 Enterprise Connection, Self-Service SSO, and SCIM.** Do NOT present those as paid "unlocks" — a customer already has them. The "Free → X" sections below list what *additionally* unlocks or increases on each plan.

## Free → B2C Essentials

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

---

## Free → B2C Professional

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

---

## Free → B2B Essentials

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

---

## Free → B2B Professional

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

## B2C Essentials → B2C Professional

FEATURES THAT UNLOCK:
✓ Enhanced Password Protection
✓ Breached Password Detection
✓ Security Center
✓ Custom Database Connections
✓ M2M Tokens (5,000 included)

FEATURES THAT REMAIN THE SAME:
= Custom Domain
= Pro MFA Factors
= Email Workflow & Branding
= Log Streaming
= Social Connections
= Organizations

---

## B2B Essentials → B2B Professional

FEATURES THAT UNLOCK:
✓ 5 Enterprise Connections (vs 3 on Essentials)
✓ Enterprise MFA Factors (included, not add-on)
✓ Enhanced Password Protection
✓ Breached Password Detection
✓ Security Center
✓ Custom Database Connections
✓ M2M Tokens (5,000 included)
✓ M2M Access for Organizations

FEATURES THAT REMAIN THE SAME:
= Custom Domain
= Organizations (unlimited)
= Pro MFA Factors
= RBAC
= Email Workflow & Branding
= Log Streaming

---

## Any Plan → Enterprise

FEATURES THAT UNLOCK:
✓ 99.99% SLA
✓ Priority Support
✓ Unlimited Custom Domains
✓ Custom rate limits & token adjustments
✓ Continuous Session Protection
✓ Prioritized Security Log Streams
✓ Private Key JWT
✓ OIDC Back-Channel Logout
✓ Custom contract terms
✓ Compliance add-ons (HIPAA/BAA, Credential Guard, Bot Detection, etc.)

---

## A4AA (Auth for AI Agents) Add-On

### What Unlocks with A4AA (50% of base price)

✓ Token Vault (unlimited)
  - Securely store, refresh, and rotate OAuth tokens for third-party API integrations
  - No secrets in code or environment variables
  - Automatic token refresh without user re-authentication

✓ CIBA (All Forms)
  - Client Initiated Backchannel Authentication
  - Async user approval for high-stakes agent actions
  - Requires Professional base plan or higher for full functionality
  - Essentials + A4AA = Token Vault + basic M2M access
  - Professional + A4AA = Token Vault + CIBA + Enhanced M2M access

✓ Enhanced M2M Token Pool
  - Higher M2M token limits for agent orchestration
  - Increased concurrent agent operations

---

## Feature Comparison: Impact Severity

### CRITICAL FEATURES (Enterprise Deal Blockers)

Custom Domain
- Impact: Brand trust + credibility
- Status: **1 included on Free**; included on Essentials and above

Organizations
- Impact: Multi-tenant data isolation
- Status: **5 on Free**; 10 on B2C Essentials/Professional; unlimited on B2B Essentials and above

Enterprise Connections (SAML/OIDC)
- Impact: Corporate SSO integration
- Status: **1 on Free**; 3 on B2B Essentials, 5 on B2B Professional (B2C: not available below Enterprise)

Log Streaming
- Impact: Compliance + audit trails
- Status: Essentials and above

MFA Enforcement
- Impact: Security + procurement requirement
- Status: Pro MFA on Essentials; Enterprise MFA included on B2B Professional

---

### HIGH-IMPACT FEATURES (Scaling/Compliance)

RBAC (Roles & Permissions)
- Impact: Team access control
- Status: B2B Essentials and above

Enhanced Password Protection
- Impact: Breach detection + credential security
- Status: Professional and above

Breached Password Detection
- Impact: Compromised credential detection
- Status: Professional and above

Token Vault (A4AA)
- Impact: Secure OAuth token management for AI agents
- Status: A4AA add-on (any plan base)

CIBA (A4AA)
- Impact: Async approval for AI agent actions
- Status: A4AA add-on (Professional base recommended)

---

### MODERATE-IMPACT FEATURES (UX/Optimization)

Custom Database Connections
- Impact: Legacy auth system migration
- Status: Professional and above

Security Center
- Impact: Centralized security dashboard
- Status: Professional and above

M2M Tokens (5,000 included)
- Impact: Machine-to-machine authentication at scale
- Status: Professional and above

Email Provider + Branding
- Impact: Branded password reset / invitation emails
- Status: Essentials and above

Per-Organization Branding
- Impact: Per-customer branded login experience
- Status: B2B Essentials and above

---

### LOW-IMPACT FEATURES (Nice-to-Have)

Bot Detection
- Impact: Automated attack prevention
- Status: Enterprise add-on

Credential Guard
- Impact: Advanced credential protection
- Status: Enterprise add-on

Adaptive MFA
- Impact: Risk-based MFA enforcement
- Status: Enterprise add-on

---

## Feature Availability by Plan (from pricing.md)

### Branding Features

| Feature | Free | B2C Essentials | B2C Professional | B2B Essentials | B2B Professional | Enterprise |
|---------|------|---|---|---|---|---|
| Custom Domains | 1 | Included | Included | Included | Included | Unlimited |
| Email Workflow | No | Yes | Yes | Yes | Yes | Yes |
| Customize Signup & Login | No | Yes | Yes | Yes | Yes | Yes |
| Per-Organization Branding | No | No | No | Yes | Yes | Yes |

### Security & Compliance Features

| Feature | Free | B2C Essentials | B2C Professional | B2B Essentials | B2B Professional | Enterprise |
|---------|------|---|---|---|---|---|
| Pro MFA Factors | No | Yes | Yes | Yes | Yes | Yes |
| Enterprise MFA Factors | No | Add-on | Included | Add-on | Included | Included |
| Enhanced Password Protection | No | No | Yes | No | Yes | Yes |
| Breached Password Detection | No | No | Yes | No | Yes | Yes |
| Security Center | No | No | Yes | No | Yes | Yes |
| Log Streaming | No | 1 stream | 1 stream | 1 stream | 1 stream | Prioritized |

### Organizations & Access Control

| Feature | Free | B2C Essentials | B2C Professional | B2B Essentials | B2B Professional | Enterprise |
|---------|------|---|---|---|---|---|
| Organizations | 5 | 10 | 10 | Unlimited | Unlimited | Unlimited |
| Enterprise Connections | 1 | Not available | Not available | 3 included | 5 included | Custom |
| RBAC (Roles & Permissions) | No | No | No | Yes | Yes | Yes |

### M2M & Developer Features

| Feature | Free | B2C Essentials | B2C Professional | B2B Essentials | B2B Professional | Enterprise |
|---------|------|---|---|---|---|---|
| M2M Tokens | No | No | 5,000 included | No | 5,000 included | Custom |
| M2M Access for Organizations | No | No | No | No | Yes | Yes |
| Custom Database Connections | No | No | Yes | No | Yes | Yes |

### AI Agent Features (A4AA Add-On)

| Feature | Free | Essentials | Professional | Enterprise |
|---------|------|---|---|---|
| Token Vault | No | Add-on | Add-on | Add-on |
| CIBA (All Forms) | No | Add-on (limited) | Add-on (full) | Add-on (full) |
| Enhanced M2M Pool | No | Add-on | Add-on | Included |

---

## Pricing Context (from pricing.md)

A4AA adds 50% to base price (rounded up):
- B2B Essentials @ 1k MAU: $300/month base + $150/month A4AA = $450/month
- B2B Professional @ 1k MAU: $800/month base + $400/month A4AA = $1,200/month

M2M Token Add-On (Professional only):
- 5,000 included
- Additional 10,000 tokens: $40/month
- Additional 20,000 tokens: $80/month