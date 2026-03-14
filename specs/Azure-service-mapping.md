# Azure Services Mapping

Maps each system capability to its Azure service and role in the architecture.

## Compute & Orchestration

| Capability | Azure Service | Notes |
|---|---|---|
| Agent Orchestration | Azure Logic Apps, Azure Durable Functions | Workflow graph, approvals, retries, state |
| Front-end Hosting | Azure App Service / Static Web Apps | React / Bot Adapter UI |

## AI & Models

| Capability | Azure Service | Notes |
|---|---|---|
| LLM / Planner Agent | Azure AI Foundry (Llama, Mistral, Phi-3) | OSS model deployment, inference endpoint |
| Content Safety | Azure AI Content Safety | Toxicity / PII filtering in Policy Agent |
| Enterprise RAG | Azure AI Search (vector + hybrid) | Citations, filters, ACL-aware retrieval |

## API & Integration

| Capability | Azure Service | Notes |
|---|---|---|
| Tool Mediation | Azure API Management (APIM) | Gateway for all tool/API calls; policies, quotas, auth |
| Messaging | Azure Service Bus, Event Grid | Async commands and events |
| Notifications | Microsoft Teams, Email (via connectors) | Approval notifications |
| LOB Systems | CRM / ERP / HRIS / Internal APIs | Backends accessed via APIM |

## Identity & Security

| Capability | Azure Service | Notes |
|---|---|---|
| Identity | Microsoft Entra ID (Azure AD) | RBAC, OAuth2, Conditional Access |
| Service Auth | Managed Identity | Per-resource least privilege |
| Secrets | Azure Key Vault | Secrets, keys, certs, rotation |

## Data & Storage

| Capability | Azure Service | Notes |
|---|---|---|
| Knowledge Store | Azure Storage (Blob) | Documents, artifacts, backups |
| State / Memory | Azure Cosmos DB | Long-term agent memory, task state |
| Search Index | Azure AI Search | Vector and hybrid indices |

## Observability

| Capability | Azure Service | Notes |
|---|---|---|
| Tracing & Logging | Azure Monitor, Application Insights | Distributed traces, action logs |
| Log Query | Log Analytics | KQL queries, alerting |
| Cost Dashboards | Azure Cost Management | Per-RG budgets and alerts |

## Governance & Compliance

| Capability | Azure Service | Notes |
|---|---|---|
| Policy Enforcement | Azure Policy | Tagging, allowed SKUs, secure configs |
| Security Monitoring | Defender for Cloud | Drift detection, recommendations |
| Data Catalog | Microsoft Purview | Lineage, data classification |

## Networking

| Capability | Azure Service | Notes |
|---|---|---|
| Network Isolation | VNet, Private Endpoints, Private DNS | PE for data/AI services |
| Egress Control | Azure Firewall | Outbound traffic filtering |
