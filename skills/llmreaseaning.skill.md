# Skill: LLM Reasoning

## Identity

| Field | Value |
|---|---|
| Name | `llm-reasoning` |
| Used By | Planner Agent |
| Backend | Azure AI Foundry OSS Model (Llama / Mistral / Phi-3) |

## Description

Provides the ability to call an OSS large language model to perform intent classification, task decomposition, and structured plan generation. This is the core "thinking" skill that powers the Planner Agent.

## Capabilities

| Capability | Description |
|---|---|
| Intent Classification | Classify a user request as retrieval, action, or mixed |
| Task Decomposition | Break a complex request into ordered atomic steps |
| Plan Generation | Produce a schema-valid JSON plan object |
| Context Awareness | Incorporate session history and user claims into reasoning |

## Model Endpoint

| Parameter | Value |
|---|---|
| Provider | Azure AI Foundry (managed endpoint) |
| Supported models | Llama 3.x, Mistral 7B/8x7B, Phi-3 |
| Authentication | Managed Identity → Foundry endpoint key (from Key Vault) |
| Protocol | HTTP REST (OpenAI-compatible chat completions API) |

## Invocation

```
POST {foundry_endpoint}/chat/completions
Headers:
  Authorization: Bearer {key_from_keyvault}
  Content-Type: application/json
Body:
  {
    "model": "{deployed_model_name}",
    "messages": [
      { "role": "system",  "content": "{system_prompt}" },
      { "role": "user",    "content": "{user_request}" }
    ],
    "temperature": 0.1,
    "max_tokens": 2048,
    "response_format": { "type": "json_object" }
  }
```

## System Prompt (Template)

```
You are a task planner for an enterprise automation system.
Given a user request and their permissions, produce a JSON execution plan.
Each step must specify: order, skill, tool (or null), description, requiresApproval (bool).
Classify the overall intent as "retrieval", "action", or "mixed".
Do not execute any actions — only plan.
```

## Output Schema

See [planner.agent.md](../agents/planner.agent.md) → Plan Object Schema.

## Error Handling

| Scenario | Action |
|---|---|
| Model timeout (> 10s) | Retry once with reduced max_tokens |
| Malformed JSON output | Log warning; retry with explicit schema reminder in prompt |
| Model unavailable | Fail step; orchestrator applies retry policy |

## Dependencies

- Azure Key Vault (endpoint key)
- Azure AI Foundry (model deployment)
- VNet / Private Endpoint (network path to Foundry)
