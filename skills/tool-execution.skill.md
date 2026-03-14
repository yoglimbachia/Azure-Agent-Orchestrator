# Skill: Tool Execution

## Identity

| Field | Value |
|---|---|
| Name | `tool-execution` |
| Used By | Executor Agent |
| Backend | Azure API Management (APIM) |

## Description

Executes tool and API calls against registered backend systems through the APIM gateway, handling authentication, correlation, retries, and response parsing.

## Capabilities

| Capability | Description |
|---|---|
| Tool Resolution | Map tool name → APIM API operation via tools registry |
| Authenticated Call | Attach OAuth2 / Managed Identity credentials |
| Correlation Propagation | Forward `correlationId` and `x-request-id` headers |
| Retry with Backoff | Exponential backoff for transient HTTP errors (429, 502, 503) |
| Response Parsing | Extract structured data from API responses |

## Tool Registry

Tools are defined in [tools/tools-registry.yaml](../tools/tools-registry.yaml). Each entry maps a logical tool name to an APIM API operation.

## Invocation Pattern

```
{HTTP_METHOD} https://{apim_gateway}/{api_path}
Headers:
  Authorization: Bearer {entra_token}
  x-correlation-id: {correlationId}
  Content-Type: application/json
Body: (per tool definition)
```

## Retry Policy

| Parameter | Value |
|---|---|
| Strategy | Exponential backoff |
| Retryable status codes | 429, 502, 503, 504 |
| Max retries | 3 |
| Initial delay | 1 second |
| Max delay | 30 seconds |
| Jitter | ± 500ms |
| Circuit breaker | 5 consecutive failures → open for 60s |

## APIM Policies (enforced at gateway)

| Policy | Purpose |
|---|---|
| `validate-jwt` | Verify Entra ID token |
| `rate-limit-by-key` | Per-user / per-app throttling (FR-9) |
| `set-header` | Remove internal headers on outbound |
| `log-to-eventhub` | Emit request/response logs |
| `cache-lookup` | Cache idempotent GET responses |
| `cors` | Allow front-end origins |

## Output Schema

```json
{
  "toolName": "itsm.createTicket",
  "status": "success",
  "durationMs": 320,
  "response": { /* parsed API result */ },
  "correlationId": "abc-123",
  "apimRequestId": "req-456"
}
```

## Error Handling

| Scenario | Action |
|---|---|
| 401 Unauthorized | Do not retry; return auth error to orchestrator |
| 429 Throttled | Retry with backoff; respect `Retry-After` header |
| 5xx Server Error | Retry up to max; then fail step |
| Timeout (> 30s) | Fail step; log timeout |
| Circuit breaker open | Fail immediately; log circuit state |

## Dependencies

- Azure API Management
- Microsoft Entra ID (token issuance)
- Azure Key Vault (backend secrets referenced by APIM named values)
- Azure Monitor / Event Hub (request logging)
