# Agent: Executor

## Identity

| Field | Value |
|---|---|
| Name | `executor` |
| Type | Sub-Agent |
| Runtime | Azure Durable Functions (activity) |
| Backend | Azure API Management (APIM gateway) |

## Purpose

Executes tool and API calls against internal and external systems through the APIM gateway, handling authentication, retries, correlation IDs, and response parsing.

## Responsibilities

1. **Tool Resolution** — Resolve the tool name from the plan step to an APIM endpoint using the tools registry.
2. **Authentication** — Attach OAuth2 bearer tokens and/or Managed Identity credentials per APIM policy.
3. **Invocation** — Execute HTTP calls to APIM-fronted APIs (CRM, ERP, HRIS, ITSM, internal APIs).
4. **Retry & Circuit Breaker** — Apply exponential backoff for transient failures; trip circuit breaker after threshold (FR-7).
5. **Correlation** — Propagate the orchestration `correlationId` through all downstream calls.
6. **Response Parsing** — Parse API responses, extract relevant data, and return structured results to the orchestrator.

## Skills Used

| Skill | Spec |
|---|---|
| Tool Execution | [tool-execution.skill.md](../skills/tool-execution.skill.md) |

## Inputs

| Input | Source | Description |
|---|---|---|
| Plan step | Orchestrator | Tool name, parameters, expected output |
| Correlation ID | Orchestrator | Run-level trace ID |
| Auth context | Orchestrator (Managed Identity / Entra token) | Credentials for APIM |

## Outputs

| Output | Description |
|---|---|
| Tool response | Parsed result from the downstream API |
| Status | `success`, `failed`, `retried` |
| Duration | Execution time in ms |
| Correlation trace | APIM request/response IDs |

## Tool Resolution

Tools are resolved via the central registry ([tools/tools-registry.yaml](../tools/tools-registry.yaml)). Each tool maps to an APIM API operation.

## Retry Policy

| Parameter | Value |
|---|---|
| Strategy | Exponential backoff |
| Max retries | 3 |
| Initial delay | 1 second |
| Max delay | 30 seconds |
| Circuit breaker threshold | 5 consecutive failures |

## APIM Integration

| Concern | APIM Policy |
|---|---|
| Authentication | `validate-jwt` (Entra ID) |
| Rate limiting | `rate-limit-by-key` |
| Header scrubbing | `set-header` (remove internal headers) |
| Request logging | `log-to-eventhub` → Log Analytics |
| Backend routing | Named backends per tool |

## Guardrails Applied

- All calls route through APIM — no direct backend access.
- Secrets for backend auth fetched from Key Vault at runtime.
- Rate limits enforced per FR-9.

## Relevant Requirements

- FR-2 (Tool / Function Calling), FR-7 (Error Handling & Retries), FR-9 (Rate Limiting)
- NFR-2 (Performance), NFR-4 (Security)
