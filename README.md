# Azure Agentic Orchestrator

Enterprise agent-based architecture for orchestrating multi-step AI workflows using Azure AI Foundry OSS models, Azure AI Search, Logic Apps / Durable Functions, API Management, and full security/governance controls.

## Project Structure

```
ob-agent-orchestrator/
│
├── projectdetail.md              ← Source functional/NFR/Azure mapping document
│
├── specs/                        ← Architecture and requirements specs
│   ├── architecture-overview.md      High-level architecture and component inventory
│   ├── functional-requirements.md    FR-1 through FR-12 with acceptance criteria
│   ├── non-functional-requirements.md NFR specs (availability, perf, security, DR)
│   ├── security-governance.md        Security, privacy, and governance plan
│   ├── azure-services-mapping.md     Azure service → capability mapping table
│   └── orchestration-flow.md         Step-by-step orchestration workflow and audit schema
│
├── agents/                       ← Agent specifications (one per agent)
│   ├── orchestrator.agent.md         Root orchestration agent
│   ├── planner.agent.md              LLM-based task planner (Foundry OSS)
│   ├── retriever.agent.md            RAG retrieval agent (Azure AI Search)
│   ├── policy.agent.md               Policy / safety enforcement agent
│   └── executor.agent.md             Tool execution agent (via APIM)
│
├── skills/                       ← Skill specifications (one per capability)
│   ├── llm-reasoning.skill.md        LLM inference and plan generation
│   ├── rag-retrieval.skill.md        Vector/hybrid search and citation
│   ├── tool-execution.skill.md       API call execution via APIM
│   ├── safety-filtering.skill.md     PII/RAI/content safety checks
│   ├── approval-workflow.skill.md    Human-in-the-loop approvals
│   └── audit-observability.skill.md  Distributed tracing and audit logging
│
├── guardrails/                   ← Safety and compliance guardrail specs
│   ├── content-safety.guardrail.md       Toxicity, PII, jailbreak prevention
│   ├── access-control.guardrail.md       RBAC, least privilege, identity rules
│   ├── rate-limiting.guardrail.md        Per-user/app quotas via APIM
│   ├── data-governance.guardrail.md      Purview, lineage, residency, retention
│   └── secrets-management.guardrail.md   Key Vault, rotation, no-secrets-in-code
│
├── tools/                        ← Tool definitions and gateway spec
│   ├── tools-registry.yaml           Central tool registry (APIM operations)
│   └── apim-gateway.tool.md          APIM gateway configuration spec
│
└── config/                       ← Runtime configuration
    └── agent.config.yaml             Agent, model, infra, and env settings
```

## How to Use These Specs

1. **Understand the architecture** — Start with `specs/architecture-overview.md` and the attached flow diagram.
2. **Review requirements** — `specs/functional-requirements.md` and `specs/non-functional-requirements.md` define what the system must do.
3. **Explore agents** — Each file in `agents/` defines a single agent's purpose, inputs, outputs, skills, and guardrails.
4. **Review skills** — Each file in `skills/` defines a capability with invocation details, schemas, and error handling.
5. **Enforce guardrails** — Each file in `guardrails/` defines safety rules with conditions and actions.
6. **Register tools** — `tools/tools-registry.yaml` is the single source of truth for callable tools.
7. **Configure** — `config/agent.config.yaml` holds runtime settings for model, infra, and environments.

## Key Azure Services

| Service | Role |
|---|---|
| Azure AI Foundry | OSS model deployment (Llama, Mistral, Phi-3) |
| Azure AI Search | Vector + hybrid RAG retrieval |
| Azure Logic Apps / Durable Functions | Orchestration engine |
| Azure API Management | Tool/API gateway |
| Microsoft Entra ID | Identity, RBAC, Managed Identity |
| Azure Key Vault | Secrets, keys, certificates |
| Azure Monitor + App Insights | Observability, audit trail |
| Azure Storage / Cosmos DB | Data stores, agent memory |
| Microsoft Purview | Data governance, lineage |
| Azure Policy + Defender | Compliance, security posture |

## Next Steps (Implementation Phase)

- [ ] Provision Azure resource groups per environment (dev / nonprod / prod)
- [ ] Deploy Foundry OSS model via Azure AI Foundry
- [ ] Create Azure AI Search index with vector + hybrid config
- [ ] Provision APIM and configure APIs from tools registry
- [ ] Deploy Logic Apps / Durable Functions orchestration skeleton
- [ ] Configure Key Vault, Managed Identities, and RBAC
- [ ] Set up Azure Monitor, App Insights, and alert rules
- [ ] Implement Bicep/Terraform modules for IaC
- [ ] CI/CD pipelines (Azure DevOps / GitHub Actions)
