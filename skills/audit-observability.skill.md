# Skill: Audit & Observability

## Identity

| Field | Value |
|---|---|
| Name | `audit-observability` |
| Used By | Orchestrator Agent (cross-cutting) |
| Backend | Azure Monitor, Application Insights, Log Analytics |

## Description

Provides end-to-end distributed tracing, structured logging, and audit-trail generation for every orchestration run. Ensures 100% action-step coverage and supports compliance queries.

## Capabilities

| Capability | Description |
|---|---|
| Distributed Tracing | Correlate all steps in a run via `correlationId` |
| Structured Logging | Emit events with consistent schema to App Insights |
| Audit Trail | Immutable per-run log (prompt, tool call, param hash, result, actor, timestamp) |
| Alerting | Trigger alerts on failure-rate spikes and latency breaches |
| Cost Dashboard | Per-resource-group cost visibility |
| Export | Exportable JSON trace per run via Log Analytics |

## Log Event Schema

| Field | Type | Description |
|---|---|---|
| `correlationId` | string | Unique run identifier |
| `stepName` | string | e.g., `plan_generated`, `policy_evaluated`, `tool_call_completed` |
| `agent` | string | Sub-agent name |
| `timestamp` | datetime (UTC) | Event time |
| `actor` | string | User principal or managed identity |
| `inputHash` | string | SHA-256 of input parameters (no raw PII) |
| `output` | object | Result summary |
| `durationMs` | int | Step duration |
| `status` | string | `success`, `failed`, `blocked`, `retried`, `approved`, `denied` |

## Standard Step Names

| Step | Emitted By |
|---|---|
| `run_started` | Orchestrator |
| `plan_generated` | Planner Agent |
| `policy_evaluated` | Policy Agent |
| `policy_blocked` | Policy Agent |
| `retrieval_completed` | Retriever Agent |
| `approval_requested` | Orchestrator |
| `approval_decision` | Orchestrator |
| `tool_call_completed` | Executor Agent |
| `tool_call_retried` | Executor Agent |
| `run_completed` | Orchestrator |
| `run_failed` | Orchestrator |

## Sample KQL Queries

### Run trace
```kql
customEvents
| where customDimensions.correlationId == "abc-123"
| project timestamp, name, customDimensions.agent, customDimensions.status, customDimensions.durationMs
| order by timestamp asc
```

### Failure rate (last 1 hour)
```kql
customEvents
| where timestamp > ago(1h)
| where customDimensions.status == "failed"
| summarize failures = count() by bin(timestamp, 5m)
| render timechart
```

### Average latency by agent
```kql
customEvents
| where timestamp > ago(24h)
| extend duration = toint(customDimensions.durationMs)
| summarize avg(duration), percentile(duration, 95) by tostring(customDimensions.agent)
```

## Alert Rules

| Alert | Condition | Severity |
|---|---|---|
| High failure rate | > 10% failures in 5-min window | Sev 2 |
| Latency breach | P95 > 30s for action flows | Sev 3 |
| Poison run detected | Run quarantined after max retries | Sev 2 |
| Approval timeout | Approval not received within SLA | Sev 4 |

## Dependencies

- Azure Application Insights (telemetry sink)
- Azure Log Analytics (query and alerting)
- Azure Monitor (alert rules and action groups)
- Azure Cost Management (budget alerts)
