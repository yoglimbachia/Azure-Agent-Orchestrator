# Guardrail: Secrets Management

## Identity

| Field | Value |
|---|---|
| Name | `secrets-management` |
| Enforced By | Policy Agent, Azure Policy, CI/CD pipeline |
| Backend | Azure Key Vault |

## Purpose

Ensure zero secrets in code, configuration, or pipeline variables. All secrets, keys, and certificates are managed centrally in Azure Key Vault with rotation and access control.

## Scope

- Application code and configuration files
- CI/CD pipeline variables
- APIM named values
- Logic Apps and Function Apps connection strings
- Agent endpoint keys (Foundry, AI Search, Content Safety)

## Rules

### R-SM-1 — No Secrets in Code

| Condition | Action |
|---|---|
| Secret, key, or connection string committed to source control | BLOCK merge — CI pipeline scan rejects |
| Secret in appsettings, environment variable, or pipeline YAML | VIOLATION — must be Key Vault reference |

### R-SM-2 — Key Vault for All Secrets

| Condition | Action |
|---|---|
| New secret required by any service | Create in Key Vault; reference via Managed Identity |
| APIM named value with plain-text secret | VIOLATION — must use Key Vault-linked named value |

### R-SM-3 — Automatic Rotation

| Condition | Action |
|---|---|
| Secret or certificate supports auto-rotation | Enable rotation policy in Key Vault |
| Manual rotation required | Documented runbook; rotation within 90 days max |

### R-SM-4 — Cryptographic Standards

| Condition | Action |
|---|---|
| Key using deprecated algorithm (e.g., RSA 1024, SHA-1) | VIOLATION — minimum RSA 2048, SHA-256 |
| Certificate validity > 12 months | VIOLATION — max 12-month validity |

### R-SM-5 — Access to Key Vault

| Condition | Action |
|---|---|
| Access via Managed Identity with scoped RBAC | ALLOWED |
| Access via shared access key or admin credentials | VIOLATION |
| Key Vault lacks Private Endpoint | VIOLATION in Prod (PE required) |

### R-SM-6 — Expiry Monitoring

| Condition | Action |
|---|---|
| Secret or certificate within 30 days of expiry | ALERT — notify ops team |
| Secret or certificate expired | CRITICAL ALERT — immediate remediation |

## CI/CD Pipeline Checks

- Pre-commit hook: `detect-secrets` or equivalent scanner.
- PR validation: scan for hardcoded secrets, connection strings, Bearer tokens.
- Deployment gate: verify Key Vault references resolve successfully.

## Related Requirements

- FR-10 (Secrets Management)
- NFR-4 (Security)
