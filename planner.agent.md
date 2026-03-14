# Agent: Planner

## Identity

| Field | Value |
|---|---|
| Name | `planner` |
| Type | Sub-Agent |
| Runtime | Azure Durable Functions (activity) |
| Model | Azure AI Foundry OSS — Llama / Mistral / Phi-3 |

## Purpose

Uses an OSS large language model deployed via Azure AI Foundry to decompose a user request into a structured, step-by-step execution plan that the orchestrator can follow.

## Responsibilities

1. **Intent Classification** — Determine the type of request (information retrieval, action execution, or mixed).
2. **Task Decomposition** — Break the request into ordered steps, each mapped to a skill or tool.
3. **Retrieval Decision** — Decide whether RAG enrichment is required before execution.
4. **Approval Flagging** — Flag steps that involve write/action on LOB systems as requiring human approval.
5. **Plan Output** — Produce a deterministic plan object consumed by the orchestrator.

## Skills Used

| Skill | Spec |
|---|---|
| LLM Reasoning | [llm-reasoning.skill.md](../skills/llm-reasoning.skill.md) |

## Inputs

| Input | Source | Description |
|---|---|---|
| User prompt | Orchestrator | Original request text |
| User claims | Orchestrator (from Entra token) | Roles, group membership |
| Session history | Orchestrator (optional) | Prior turns for context |

## Outputs

| Output | Description |
|---|---|
| Plan object | JSON: ordered list of steps with `skill`, `tool`, `requiresApproval`, `requiresRetrieval` flags |
| Intent label | Classification string (e.g., `retrieval`, `action`, `mixed`) |

## Plan Object Schema

```json
{
  "intent": "action",
  "steps": [
    {
      "order": 1,
      "skill": "rag-retrieval",
      "tool": null,
      "description": "Retrieve relevant context from enterprise knowledge base",
      "requiresApproval": false
    },
    {
      "order": 2,
      "skill": "tool-execution",
      "tool": "itsm.createTicket",
      "description": "Create a support ticket in the ITSM system",
      "requiresApproval": true
    }
  ]
}
```

## Model Configuration

| Parameter | Value |
|---|---|
| Deployment | Azure AI Foundry managed endpoint |
| Models supported | Llama 3.x, Mistral, Phi-3 |
| Temperature | 0.1 (deterministic planning) |
| Max tokens | 2048 |
| System prompt | Defined in skills/llm-reasoning.skill.md |

## Guardrails Applied

- Policy Agent validates the plan **after** generation (no self-validation).
- Model output is parsed and schema-validated before use.

## Relevant Requirements

- FR-1 (Agentic Workflows), FR-3 (Safety — plan validation downstream)
- NFR-2 (Performance — model latency < 3s P50)
