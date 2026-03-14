# Tool Spec: APIM Gateway

## Identity

| Field | Value |
|---|---|
| Name | `apim-gateway` |
| Type | Infrastructure Tool |
| Azure Service | Azure API Management |

## Purpose

All tool and API calls from agents are mediated through APIM. This spec defines how APIM is configured to front backend tools, enforce security policies, and provide observability.

## Architecture Role

```
Executor Agent
    │
    ▼
┌─────────────────────────┐
│  Azure API Management   │
│  ─ JWT validation       │
│  ─ Rate limiting        │
│  ─ Header scrubbing     │
│  ─ Request logging      │
│  ─ Backend routing      │
└────────┬────────────────┘
         │
    ┌────┴────┬─────────┬──────────┬─────────┐
    ▼         ▼         ▼          ▼         ▼
      Internal APIs
```

## APIM Products

| Product | Description | Rate Limit | Target Consumers |
|---|---|---|---|
| `agent-standard` | Standard agent access | 30 calls/min | Default for all agents |
| `agent-elevated` | Elevated access for approved actions | 60 calls/min | Executor after approval |

## APIM APIs

| API Name | Backend | Base Path | Auth |
|---|---|---|---|
| `itsm-api` | ITSM system | /itsm | OAuth2 (Entra) |
| `crm-api` | CRM system | /crm | OAuth2 (Entra) |
| `erp-api` | ERP system | /erp | OAuth2 (Entra) |
| `hris-api` | HRIS system | /hris | OAuth2 (Entra) |
| `messaging-api` | Teams / Email connectors | /messaging | OAuth2 (Entra) |
| `internal-api` | Internal LOB APIs | /internal | OAuth2 (Entra) |

## Global Policies

### Inbound

| Policy | Purpose |
|---|---|
| `validate-jwt` | Verify Entra ID token; check audience and issuer |
| `rate-limit-by-key` | Per-user rate limiting (FR-9) |
| `quota-by-key` | Per-subscription daily quota |
| `set-header` | Add `x-correlation-id`; remove internal headers |
| `ip-filter` (Prod) | Restrict to VNet / known IPs |

### Backend

| Policy | Purpose |
|---|---|
| `authentication-managed-identity` | Attach MI token for backend calls |
| `forward-request` | Forward to named backend |

### Outbound

| Policy | Purpose |
|---|---|
| `set-header` | Remove backend-internal headers |
| `find-and-replace` | Mask sensitive values in responses |

### On-Error

| Policy | Purpose |
|---|---|
| `return-response` | User-friendly error message |
| `log-to-eventhub` | Emit error to Log Analytics |

## Named Values (Key Vault Linked)

| Named Value | Key Vault Secret | Purpose |
|---|---|---|
| `itsm-backend-url` | `kv-itsm-url` | Backend base URL |
| `crm-backend-url` | `kv-crm-url` | Backend base URL |
| `apim-appinsights-key` | `kv-appinsights-ikey` | Instrumentation key |

## Networking

| Environment | Configuration |
|---|---|
| Dev | External APIM, public access |
| Non-Prod | Internal APIM, VNet-integrated |
| Prod | Internal APIM, Private Endpoints, VNet-integrated |

## Related Requirements

- FR-2 (Tool Calling), FR-9 (Rate Limiting), FR-10 (Secrets)
- NFR-4 (Security), NFR-3 (Scalability)
