# Architecture Overview

## 1. Summary

This system is an **enterprise agentic AI orchestrator** built on Azure. It receives natural-language requests from end users, decomposes them into plans, enforces safety and policy guardrails, retrieves enterprise knowledge via RAG, executes tool/API calls through a governed gateway, and returns auditable results — all without embedding secrets or bypassing access controls.

## 2. Layers

```
┌─────────────────────────────────────────────────────────────────┐
│  End User  (Portal / WebChat / App)                             │
└──────────────────────────┬──────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│  Front-end UI  (React / Web App / Bot Adapter)                  │
└──────────────────────────┬──────────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────────┐
│  Agent Orchestration Engine                                     │
│  Azure Logic Apps / Durable Functions                           │
│  Routes to sub-agents, manages state, retries, approvals        │
└──┬──────────┬──────────┬──────────┬─────────────────────────────┘
   │          │          │          │
   ▼          ▼          ▼          ▼
 Policy    Planner    Executor   Retriever
 Agent     Agent      Agent      Agent
```

## 3. Component Inventory

| Component | Azure Service | Role |
|---|---|---|
| Orchestration Engine | Logic Apps / Durable Functions | Workflow coordination, state, retries |
| Planner Agent | Azure AI Foundry OSS (Llama / Mistral / Phi-3) | LLM-based task decomposition and planning |
| Retriever Agent | Azure AI Search (vector + hybrid) | RAG retrieval, data enrichment, citations |
| Policy / Safety Agent | Azure AI Content Safety + custom logic | PII filtering, RAI checks, access validation |
| Executor Agent | Azure API Management (APIM) | Tool/API invocation, retries, correlation |
| Identity | Microsoft Entra ID | RBAC, OAuth2, Managed Identity |
| Secrets | Azure Key Vault | Secrets, keys, certificates, rotation |
| Observability | Azure Monitor + Application Insights | Tracing, logging, audit trail |
| Data Stores | Azure Storage (Blob) / Cosmos DB | Knowledge, logs, long-term memory |
| Messaging | Service Bus / Event Grid / Teams / Email | Async events, notifications |
| Governance | Azure Policy + Defender for Cloud + Purview | Compliance, lineage, data catalog |
| Networking | VNet, Private Endpoints, Firewall | Egress control, PE for data/AI |
| External Systems | CRM / ERP / HRIS / Internal APIs | Line-of-business backends |

## 4. Data Flow (Happy Path)

1. End user submits a natural-language request through the front-end UI.
2. The UI forwards the request to the **Orchestration Engine** (Logic Apps / Durable Functions).
3. The engine invokes the **Planner Agent**, which calls the Foundry OSS model to classify intent and produce a step-by-step execution plan.
4. The engine invokes the **Policy / Safety Agent** to validate the plan against RAI, PII, and RBAC policies.
5. *(If the plan requires enterprise knowledge)* The engine invokes the **Retriever Agent** to perform a vector/hybrid search against Azure AI Search and enrich context.
6. The engine invokes the **Executor Agent** to execute tool/API calls through APIM, passing OAuth2 tokens and managed-identity credentials.
7. Results flow back through the engine, which logs every step to Azure Monitor / App Insights.
8. The final response (with citations and audit trail) is returned to the user.

## 5. Environment Strategy

| Environment | Purpose | Isolation |
|---|---|---|
| Dev | Developer inner-loop, experimentation | Dedicated RG + identities |
| Non-Prod | Integration testing, UAT, staging | Dedicated RG + identities |
| Prod | Live traffic | Dedicated RG + identities, private endpoints enforced |

## 6. Cross-Cutting Concerns

- **Authentication**: Entra ID for all human and service identities; Managed Identity for compute.
- **Secrets**: All secrets/certs in Key Vault; no secrets in code or config.
- **Networking**: Private Endpoints for AI Search, Storage, Cosmos DB; egress via Firewall.
- **Observability**: 100% action-step tracing; alerts on failure rates and latency.
- **Governance**: Azure Policy enforces tagging, allowed SKUs, and secure configs; Defender for Cloud monitors drift.
