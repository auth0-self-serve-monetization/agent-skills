# Remediation Command Map (gated apply)

Used by **Phase 7 — Optional gated apply**. For each approved fix, build the command using a first-class subcommand when one exists; otherwise fall back to `auth0 api <method> <path>` with an explicit JSON payload, where `<method>` is the HTTP method shown for that resource in the mapping table below (PATCH / PUT / POST — not always PATCH). Run `auth0 <cmd> --help` first if unsure of flag names.

## Per-item flow (mandatory for every item)

1. **Build the command(s)** — multiple are fine (e.g. callbacks + origins + grant types may need 1–3 calls).
2. **Show the diff** — fetch current value (`auth0 api get ...`) and print current-vs-proposed side by side. Never show only the proposed payload.
3. **Print the exact command(s)** about to run, including JSON payloads.
4. **Ask via `AskUserQuestion`:** Implement now / Queue / Skip. Proceed only on "Implement now".
5. **Execute via Bash**, capture stdout/stderr.
6. **Verify** by re-fetching the same resource and confirming the field changed. Never claim success on exit code alone.
7. **On failure, do not retry destructively** — surface the error and ask how to proceed.

## Command-shape mapping

| Category | First-class command (if available) | Fallback `auth0 api` path |
|---|---|---|
| Tenant settings | `auth0 tenants settings update` | `PATCH /api/v2/tenants/settings` |
| Brute-force protection | `auth0 protection brute-force-protection update` | `PATCH /api/v2/attack-protection/brute-force-protection` |
| Breached password detection | `auth0 protection breached-password-detection update` | `PATCH /api/v2/attack-protection/breached-password-detection` |
| Suspicious IP throttling | `auth0 protection suspicious-ip-throttling update` | `PATCH /api/v2/attack-protection/suspicious-ip-throttling` |
| Branding (universal login, colors, logo) | `auth0 branding update` / `auth0 universal-login update` | `PATCH /api/v2/branding` |
| Custom domains | `auth0 domains create` / `update` | `POST/PATCH /api/v2/custom-domains` |
| Connections (DB, social, enterprise) | `auth0 connections update <id>` | `PATCH /api/v2/connections/{id}` (full options blob) |
| APIs / resource servers | `auth0 apis update` | `PATCH /api/v2/resource-servers/{id}` |
| Apps / clients (callbacks, origins, grant types, refresh tokens) | `auth0 apps update` | `PATCH /api/v2/clients/{id}` |
| Roles | `auth0 roles update` | `PATCH /api/v2/roles/{id}` |
| Actions | `auth0 actions create/update/deploy` | `POST/PATCH /api/v2/actions/actions` |
| Log streams | `auth0 logs streams create/update` | `POST/PATCH /api/v2/log-streams` |
| Email provider/templates | `auth0 email provider update` / `auth0 email templates update` | `PATCH /api/v2/emails/provider` / `/api/v2/email-templates/{name}` |
| MFA factor toggles | (none) | `PUT /api/v2/guardian/factors/{factor}` |
| MFA policies | (none) | `PUT /api/v2/mfa/policies` |
| Prompts customization | (none) | `PUT /api/v2/prompts/{prompt}/custom-text/{lang}` |
| Network ACLs | (none) | `POST/PATCH /api/v2/network-acls` |
| Auth0 Organizations | `auth0 orgs create/update` | `POST/PATCH /api/v2/organizations` |

The fallback is `auth0 api <method> <path> --data '<json>'`, where `<method>` matches the table above: `patch` for most resources, `put` for MFA factor toggles, MFA policies, and Prompts customization, and `post` when creating a resource (custom domains, log streams, network ACLs, organizations). Note that `auth0 api` defaults to GET without `--data` and POST with `--data`, so always state the method explicitly.

## MFA safety rule (do not lock users out)

**Enabling an MFA factor ≠ enforcing MFA, and you must not enforce until a factor can actually deliver.** Before enforcing MFA: verify ≥1 enabled factor can deliver (SMS provider configured / email domain verified / WebAuthn available). Recommend a phishing-resistant factor (WebAuthn) as the safe default. Keep "enable factor" and "enforce policy" as distinct, clearly-labeled steps.

## Fix dependencies

Some fixes have prerequisites — Email MFA needs SMTP first; enforcing MFA needs a live factor; custom domain needs DNS. Detect/declare prerequisites and order (or warn) accordingly; don't apply a fix whose prerequisite isn't met.

## Never without explicit confirmation

These are blocked unless the user types the action verbatim:

- `auth0 logout`
- `auth0 tenants delete`
- `auth0 apps delete` (any app)
- `auth0 connections delete`
- `auth0 api delete <anything>`
