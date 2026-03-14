# Guardrail: Rate Limiting & Quotas

## Identity

| Field | Value |
|---|---|
| Name | `rate-limiting` |
| Enforced By | Azure API Management (APIM policies) |
| Backend | APIM rate-limit and quota policies |

## Purpose

Prevent abuse, ensure fair usage, and protect downstream systems from overload by enforcing per-user and per-application rate limits and quotas at the API gateway layer.

## Scope

- All tool/API calls routed through APIM
- LLM inference calls (if routed through APIM)
- Search queries (if routed through APIM)

## Rules

### R-RL-1 — Per-User Rate Limit

| Parameter | Value |
|---|---|
| Window | 60 seconds |
| Max calls | 30 (configurable per product/tier) |
| Action on breach | 429 Too Many Requests with `Retry-After` header |

### R-RL-2 — Per-Application Quota

| Parameter | Value |
|---|---|
| Window | 24 hours |
| Max calls | 10,000 (configurable per subscription) |
| Action on breach | 429 with user-friendly message |

### R-RL-3 — LLM Token Budget

| Parameter | Value |
|---|---|
| Window | Per orchestration run |
| Max tokens | 50,000 (prompt + completion combined) |
| Action on breach | Fail plan step; log `token_budget_exceeded` |

### R-RL-4 — Concurrent Request Limit

| Parameter | Value |
|---|---|
| Max concurrent per user | 5 |
| Action on breach | 429 with backpressure message |

## APIM Policy Fragment

```xml
<rate-limit-by-key
    calls="30"
    renewal-period="60"
    counter-key="@(context.Request.Headers.GetValueOrDefault("Authorization","").AsJwt()?.Subject)" />
<quota-by-key
    calls="10000"
    renewal-period="86400"
    counter-key="@(context.Subscription.Id)" />
```

## Monitoring

- Rate-limit breaches are emitted to Log Analytics.
- Dashboard shows top consumers and throttle frequency.
- Alert if any single user exceeds 80% of quota.

## Related Requirements

- FR-9 (Rate Limiting & Quotas)
- NFR-3 (Scalability)
