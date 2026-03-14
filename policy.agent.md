# Agent: Policy / Safety

## Identity

| Field | Value |
|---|---|
| Name | `policy` |
| Type | Sub-Agent |
| Runtime | Azure Durable Functions (activity) |
| Backend | Azure AI Content Safety + custom policy logic |

## Purpose

Enforces Responsible AI (RAI), PII filtering, RBAC validation, and organizational policies **before** any tool execution or data retrieval. Acts as the safety gate in every orchestration run.

## Responsibilities

1. **Content Safety** — Check user prompts and generated plans against toxicity, hate, violence, and self-harm categories via Azure AI Content Safety.
2. **PII Detection & Redaction** — Identify and redact or mask personally identifiable information before data crosses trust boundaries.
3. **RBAC Validation** — Verify the user's Entra ID claims against the permissions required by each planned step.
4. **Policy Rules** — Evaluate organizational rules (e.g., payment limits, allowed tool scopes, geo restrictions).
5. **Approval Flagging** — Confirm or override approval requirements flagged by the Planner.
6. **Block with Rationale** — If any check fails, return a user-visible rationale and block execution.

## Skills Used

| Skill | Spec |
|---|---|
| Safety Filtering | [safety-filtering.skill.md](../skills/safety-filtering.skill.md) |

## Inputs

| Input | Source | Description |
|---|---|---|
| Plan object | Orchestrator (from Planner) | Steps to validate |
| User prompt | Orchestrator | Original request for content check |
| User claims | Orchestrator (from Entra token) | Roles, groups |
| Policy rules | Configuration / Key Vault | Org-specific rules |

## Outputs

| Output | Description |
|---|---|
| Verdict | `ALLOW`, `BLOCK`, or `ALLOW_WITH_CONDITIONS` |
| Rationale | Human-readable explanation (if blocked) |
| Redacted prompt | Prompt with PII masked (if applicable) |
| Modified plan | Plan with approval flags confirmed |

## Policy Categories

| Category | Check | Source |
|---|---|---|
| Content Safety | Toxicity, hate, violence, self-harm | Azure AI Content Safety |
| PII | Names, emails, phone numbers, IDs | Custom regex + AI Content Safety |
| RBAC | User roles vs. required permissions | Entra ID token claims |
| Business Rules | Payment limits, tool scope, geo | Configuration |
| Secret Leakage | API keys, connection strings in prompt | Pattern matching |

## Guardrail Specs

- [content-safety.guardrail.md](../guardrails/content-safety.guardrail.md)
- [access-control.guardrail.md](../guardrails/access-control.guardrail.md)
- [secrets-management.guardrail.md](../guardrails/secrets-management.guardrail.md)

## Behaviour on Block

```
Policy Agent returns BLOCK
     │
     ├─ Orchestrator logs: policy_blocked (with rationale hash)
     ├─ Orchestrator returns user-visible rationale
     └─ Run ends — no downstream agents invoked
```

## Relevant Requirements

- FR-3 (Safety Guardrails), FR-5 (Approvals), FR-10 (Secrets), FR-11 (RBAC)
- NFR-4 (Security), NFR-5 (Privacy)
