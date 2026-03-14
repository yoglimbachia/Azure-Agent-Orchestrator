# Guardrail: Content Safety

## Identity

| Field | Value |
|---|---|
| Name | `content-safety` |
| Enforced By | Policy / Safety Agent |
| Backend | Azure AI Content Safety |

## Purpose

Prevent harmful, toxic, or inappropriate content from being processed or returned by any agent in the system.

## Scope

- User prompts (inbound)
- LLM-generated plans and responses (outbound)
- Retrieved documents before inclusion in responses

## Rules

### R-CS-1 — Toxic Content Blocking

| Condition | Action |
|---|---|
| Any content category rated HIGH (hate, violence, self-harm, sexual) | BLOCK — return user-visible rationale |
| Any content category rated MEDIUM | WARN — log and flag for review; allow with caution |

### R-CS-2 — Jailbreak Detection

| Condition | Action |
|---|---|
| Jailbreak attempt detected | BLOCK — do not forward to any downstream agent |

### R-CS-3 — PII in Prompts

| Condition | Action |
|---|---|
| PII entities detected in user prompt | REDACT — mask before forwarding to LLM or tools |

### R-CS-4 — PII in Responses

| Condition | Action |
|---|---|
| PII entities detected in LLM output | REDACT — mask before returning to user |

### R-CS-5 — Secret Leakage

| Condition | Action |
|---|---|
| API key, connection string, or token pattern in prompt or output | BLOCK — do not process further |

## Entity Types Monitored

PersonName, Email, Phone, NationalId, CreditCard, IBAN, APIKey, ConnectionString, BearerToken

## Audit

- Every content safety evaluation is logged with `stepName: policy_evaluated`.
- Blocked requests include the rationale hash (no raw content) in the audit trail.

## Related Requirements

- FR-3 (Safety Guardrails)
- NFR-5 (Privacy)
