# Orchestration Flow

## Overview

The orchestration engine (Logic Apps / Durable Functions) coordinates all sub-agents through a deterministic workflow graph. Each run produces a full audit trace.

## Primary Flow

```
User Request
     │
     ▼
┌─────────────────────────────────┐
│  1. Receive & Validate Request  │  Orchestration Engine
│     - Authenticate via Entra ID │
│     - Create correlation ID     │
│     - Log: run_started          │
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│  2. Plan                        │  → Planner Agent
│     - Call Foundry OSS model    │
│     - Classify intent           │
│     - Produce step-by-step plan │
│     - Log: plan_generated       │
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│  3. Policy & Safety Check       │  → Policy / Safety Agent
│     - RAI content check         │
│     - PII redaction             │
│     - RBAC validation           │
│     - Log: policy_evaluated     │
│     - If BLOCKED → respond      │
│       with rationale and exit   │
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│  4. Retrieve (conditional)      │  → Retriever Agent
│     - Vector / hybrid search    │
│       on Azure AI Search        │
│     - Enrich context with       │
│       citations                 │
│     - Log: retrieval_completed  │
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│  5. Approval (conditional)      │  → Orchestration Engine
│     - If plan includes          │
│       sensitive action →        │
│       pause workflow            │
│     - Send approval request     │
│       (Teams / Email)           │
│     - Wait for approve / deny   │
│     - Log: approval_decision    │
│     - If DENIED → exit          │
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│  6. Execute                     │  → Executor Agent
│     - Call tools via APIM       │
│     - OAuth2 + Managed Identity │
│     - Handle retries (exp.      │
│       backoff)                  │
│     - Log: tool_call_completed  │
└────────────┬────────────────────┘
             │
             ▼
┌─────────────────────────────────┐
│  7. Respond & Audit             │  Orchestration Engine
│     - Assemble final response   │
│     - Include citations if RAG  │
│     - Write full trace to       │
│       Monitor / App Insights    │
│     - Log: run_completed        │
└─────────────────────────────────┘
```

## Error Handling Flow

```
Any step failure
     │
     ├─ Transient? → Retry with exponential backoff (max 3)
     │                  └─ Still failing? → mark as poison, quarantine
     │
     └─ Non-transient? → Log error, notify operator, abort run
```

## Audit Log Schema (per step)

| Field | Type | Description |
|---|---|---|
| `correlationId` | string | Unique run identifier |
| `stepName` | string | e.g., `plan_generated`, `tool_call_completed` |
| `agent` | string | Sub-agent that executed the step |
| `timestamp` | datetime | UTC timestamp |
| `actor` | string | User or managed identity |
| `inputHash` | string | SHA-256 hash of input parameters |
| `output` | object | Result summary (no PII) |
| `durationMs` | int | Step duration in milliseconds |
| `status` | string | `success`, `failed`, `blocked`, `retried` |

## Branching Rules

| Condition | Action |
|---|---|
| Plan is retrieval-only (no side effects) | Skip approval step |
| Plan includes write/action on LOB system | Require approval (FR-5) |
| Policy agent returns BLOCKED | Return rationale; skip execute |
| Retrieval returns zero results | Planner re-plans or returns "no data" |
