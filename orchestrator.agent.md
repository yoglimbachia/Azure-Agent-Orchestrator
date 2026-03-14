# Agent: Orchestrator

## Identity

| Field | Value |
|---|---|
| Name | `orchestrator` |
| Type | Root / Coordinator |
| Runtime | Azure Logic Apps / Azure Durable Functions |
| Model | None (deterministic workflow engine) |

## Purpose

The root orchestration agent that receives end-user requests, coordinates all sub-agents, manages workflow state, retries, approvals, and produces a complete audit trace for every run.

## Responsibilities

1. **Receive** — Accept user requests from the front-end UI; authenticate via Entra ID; assign a correlation ID.
2. **Delegate** — Route to sub-agents in the correct order: Planner → Policy → Retriever (conditional) → Approval (conditional) → Executor.
3. **State Management** — Persist workflow state across steps using Durable Functions or Logic Apps run history.
4. **Error Handling** — Apply retry policies (exponential backoff); detect and quarantine poison runs (FR-7).
5. **Approvals** — Pause workflow for human approval; send notifications via Teams / Email; resume or abort based on decision (FR-5).
6. **Audit** — Log every step (prompt, tool call, parameter hash, result, actor, timestamp) to Azure Monitor / App Insights (FR-6).
7. **Response** — Assemble the final response (with citations if RAG was used) and return to the user.

## Sub-Agents

| Sub-Agent | When Invoked | Spec |
|---|---|---|
| Planner | Every request | [planner.agent.md](planner.agent.md) |
| Policy / Safety | Every request (after Plan) | [policy.agent.md](policy.agent.md) |
| Retriever | When plan requires knowledge / RAG | [retriever.agent.md](retriever.agent.md) |
| Executor | When plan includes tool/API calls | [executor.agent.md](executor.agent.md) |

## Inputs

| Input | Source | Description |
|---|---|---|
| User prompt | Front-end UI | Natural-language request |
| User identity | Entra ID token | RBAC claims, group memberships |
| Session context | Cosmos DB (optional) | Prior conversation state |

## Outputs

| Output | Destination | Description |
|---|---|---|
| Response | Front-end UI | Final answer with citations |
| Audit trace | Azure Monitor / App Insights | Full step-by-step log |
| Approval request | Teams / Email | Notification for human approval |

## Configuration

See [config/agent.config.yaml](../config/agent.config.yaml) for runtime settings (max iterations, retry policy, sub-agent bindings).

## Relevant Requirements

- FR-1 (Agentic Workflows), FR-5 (Approvals), FR-6 (Auditability), FR-7 (Error Handling), FR-8 (Multi-Environment)
- NFR-1 (Availability), NFR-2 (Performance), NFR-7 (Observability)
