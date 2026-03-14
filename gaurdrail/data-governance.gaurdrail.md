# Guardrail: Data Governance

## Identity

| Field | Value |
|---|---|
| Name | `data-governance` |
| Enforced By | Policy Agent, Azure Policy, Microsoft Purview |
| Backend | Microsoft Purview, Azure Policy, Azure DevOps |

## Purpose

Ensure all data sources, prompts, and derived artifacts are cataloged, classified, and governed with visible lineage. Enforce data residency, retention, and change-management policies.

## Scope

- Enterprise knowledge indexed in Azure AI Search
- Agent memory stored in Cosmos DB
- Prompt and response logs in App Insights
- Derived artifacts (plans, tool results)

## Rules

### R-DG-1 — Data Catalog Registration

| Condition | Action |
|---|---|
| New data source added to AI Search or Cosmos DB | MUST be registered in Microsoft Purview |
| Source not in Purview catalog | BLOCK indexing until registered |

### R-DG-2 — Data Classification

| Condition | Action |
|---|---|
| Source contains PII or sensitive data | Label with appropriate sensitivity classification |
| Classification missing | Flag for review; do not promote to Prod |

### R-DG-3 — Lineage Tracking

| Condition | Action |
|---|---|
| Data flows from source → index → agent response | Lineage must be visible in Purview for at least one workflow |
| Lineage broken or missing | Alert data governance team |

### R-DG-4 — Data Residency

| Condition | Action |
|---|---|
| Data stored outside approved Azure regions | VIOLATION — Azure Policy denies deployment |
| Cross-region replication without approval | VIOLATION |

### R-DG-5 — Retention & Purge

| Condition | Action |
|---|---|
| Data exceeds retention policy (e.g., 90 days for logs) | Auto-purge or archive |
| PII stored beyond purpose-bound period | VIOLATION |

### R-DG-6 — Change Management for Production

| Condition | Action |
|---|---|
| Index schema change, new data source, or model update for Prod | Requires Azure DevOps PR + ServiceNow ("SNOW on ADO") approval |
| Direct production change without CM | VIOLATION |

## Access Management

- Data access governed by Entra ID Access Packages.
- Access requests go through Purview / Entra Access Reviews.

## Related Requirements

- FR-12 (Data Residency & Lineage)
- NFR-5 (Privacy), NFR-6 (Compliance)
