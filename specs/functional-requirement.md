# Functional Requirements

## FR-1 — Agentic Workflows

**Description**: The system must execute multi-step workflows (plan → retrieve → act) with branching and human approvals.

**Acceptance Criteria**:
- Orchestration produces an end-to-end action log per run.
- Approvals can pause and resume a workflow without data loss.
- Branching paths (e.g., retrieval-only vs. action flows) are supported.

**Mapped Components**: Orchestration Engine (Logic Apps / Durable Functions), all sub-agents.

---

## FR-2 — Tool / Function Calling

**Description**: Agents must call registered tools via APIM using OAuth2 / Entra ID and managed identities.

**Acceptance Criteria**:
- A sample tool call (e.g., ITSM ticket creation) works end-to-end via APIM with RBAC.
- No embedded secrets in any tool call path.
- Tools are discoverable via a central registry.

**Mapped Components**: Executor Agent, Azure API Management, Microsoft Entra ID, Key Vault.

---

## FR-3 — Safety Guardrails

**Description**: Pre-action checks (policy, PII/secret redaction) and contextual allow/deny before any tool execution.

**Acceptance Criteria**:
- Unsafe prompts or actions are blocked with a user-visible rationale.
- PII is redacted before data leaves the safety boundary.
- Guardrail decisions are logged in the audit trail.

**Mapped Components**: Policy / Safety Agent, Azure AI Content Safety.

**Guardrail Specs**: [content-safety.guardrail.md](../guardrails/content-safety.guardrail.md), [access-control.guardrail.md](../guardrails/access-control.guardrail.md).

---

## FR-4 — Enterprise Memory (RAG)

**Description**: Answers and actions must be grounded on enterprise data with citations.

**Acceptance Criteria**:
- Chat responses surface links/IDs to Azure AI Search items.
- Access is denied if the user lacks RBAC for the underlying data.
- Hybrid (vector + keyword) retrieval is supported.

**Mapped Components**: Retriever Agent, Azure AI Search, Azure Storage / Cosmos DB.

---

## FR-5 — Approvals (Human-in-the-Loop)

**Description**: Sensitive actions require one or more approvers before execution.

**Acceptance Criteria**:
- At least one approval policy is defined and enforced.
- Approver receives a notification (Teams / Email) with a link to approve or deny.
- The orchestration engine honors the approval decision and resumes or aborts the workflow.

**Mapped Components**: Orchestration Engine, Messaging (Teams / Email), Logic Apps connectors.

---

## FR-6 — Auditability

**Description**: Every orchestration step is logged — prompt, tool call, parameters hash, result, actor, and timestamp.

**Acceptance Criteria**:
- All actions are queryable in Log Analytics.
- An exportable JSON trace per run is available.
- Traces include correlation IDs across all sub-agents.

**Mapped Components**: Azure Monitor, Application Insights, Log Analytics.

---

## FR-7 — Error Handling & Retries

**Description**: Transient failures use exponential backoff; poison-message detection quarantines failing runs.

**Acceptance Criteria**:
- Retries and final disposition are visible in the run trace.
- Poison runs are quarantined and surfaced for manual review.
- Circuit breaker patterns prevent cascading failures.

**Mapped Components**: Orchestration Engine (Durable Functions retry policies), Executor Agent.

---

## FR-8 — Multi-Tenant / Multi-Environment Isolation

**Description**: Dev, Non-Prod, and Prod environments are fully isolated.

**Acceptance Criteria**:
- Resource groups and identities are environment-scoped.
- No cross-environment data leaks.
- Environment promotion follows IaC (Bicep/Terraform) + CI/CD pipelines.

**Mapped Components**: Azure Policy, Resource Groups, Managed Identities.

---

## FR-9 — Rate Limiting & Quotas

**Description**: Per-user and per-application quotas enforced at the API gateway layer.

**Acceptance Criteria**:
- APIM policies throttle excess calls and return user-friendly messages.
- Quota metrics are visible in dashboards.

**Mapped Components**: Azure API Management (policies), Azure Monitor.

---

## FR-10 — Secrets Management

**Description**: No secrets in code or configuration files; all secrets managed centrally.

**Acceptance Criteria**:
- All secrets, certificates, and keys are stored in Azure Key Vault.
- Automatic rotation is enabled where supported.
- APIM named values reference Key Vault.

**Mapped Components**: Azure Key Vault, APIM, Managed Identities.

**Guardrail Spec**: [secrets-management.guardrail.md](../guardrails/secrets-management.guardrail.md).

---

## FR-11 — RBAC & Least Privilege

**Description**: All runtime identities use managed identities with minimal roles.

**Acceptance Criteria**:
- Access reviews pass with no over-privileged identities.
- No owner-level roles on runtime identities.
- Conditional access policies apply to admin accounts.

**Mapped Components**: Microsoft Entra ID, Managed Identities, Azure Policy.

**Guardrail Spec**: [access-control.guardrail.md](../guardrails/access-control.guardrail.md).

---

## FR-12 — Data Residency & Lineage

**Description**: All data sources are cataloged with visible lineage.

**Acceptance Criteria**:
- Datasets are registered in Microsoft Purview.
- Lineage is visible for at least one end-to-end workflow.
- Change management integrates with Azure DevOps + ServiceNow before production promotions.

**Mapped Components**: Microsoft Purview, Azure DevOps, Azure Policy.

**Guardrail Spec**: [data-governance.guardrail.md](../guardrails/data-governance.guardrail.md).
