# Guardrail: Access Control

## Identity

| Field | Value |
|---|---|
| Name | `access-control` |
| Enforced By | Policy / Safety Agent, Orchestrator |
| Backend | Microsoft Entra ID, Azure RBAC |

## Purpose

Ensure every agent action respects least-privilege access, RBAC roles, and identity-based authorization. No runtime identity may hold excessive permissions.

## Scope

- User-to-orchestrator authentication
- Agent-to-tool authorization (via APIM)
- Agent-to-data authorization (AI Search ACL, Storage, Cosmos DB)
- Admin access to infrastructure

## Rules

### R-AC-1 — User Authentication

| Condition | Action |
|---|---|
| Request without valid Entra ID token | REJECT — 401 Unauthorized |
| Token expired or invalid signature | REJECT — 401 Unauthorized |

### R-AC-2 — User Authorization per Step

| Condition | Action |
|---|---|
| User lacks required role for a planned step | BLOCK step — return "insufficient permissions" |
| User group claims don't match AI Search ACL | FILTER — return only authorized results |

### R-AC-3 — Service Identity — Managed Identity Only

| Condition | Action |
|---|---|
| Service authenticates with embedded secret | VIOLATION — must use Managed Identity |
| Runtime identity has Owner or Contributor on subscription | VIOLATION — maximum allowed: scoped Reader/Contributor per resource |

### R-AC-4 — APIM JWT Validation

| Condition | Action |
|---|---|
| API call missing or invalid JWT | APIM rejects with 401 |
| JWT audience mismatch | APIM rejects with 403 |

### R-AC-5 — Admin Access

| Condition | Action |
|---|---|
| Admin access without MFA / Conditional Access | VIOLATION |
| Permanent privileged role assignment | VIOLATION — use PIM (Privileged Identity Management) |

### R-AC-6 — Periodic Access Reviews

| Condition | Action |
|---|---|
| No access review in last 90 days | ALERT — trigger Entra Access Review |

## Audit

- All auth/authz failures are logged with correlation ID.
- Access review status tracked in compliance dashboard.

## Related Requirements

- FR-2 (Tool Calling — RBAC), FR-11 (RBAC & Least Privilege)
- NFR-4 (Security)
