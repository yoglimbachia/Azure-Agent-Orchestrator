# AGENTS.md — System Agent Card
# Azure Agentic Orchestrator

> This is the top-level behavioral contract for the agentic system. It defines scope, constraints, ownership, and operating boundaries for all agents in this project.

---

## System Identity

| Field | Value |
|---|---|
| System Name | Azure Agentic Orchestrator |
| Version | 0.1.0 |
| Owner | Platform Engineering Team |
| Status | Spec / Pre-implementation |
| Last Updated | March 2026 |

---

## Purpose

This system orchestrates enterprise AI workflows using a multi-agent architecture deployed on Azure. It accepts natural-language requests from authenticated users, decomposes them into plans, enforces safety and compliance guardrails, retrieves grounded knowledge, and executes tool calls against internal and external systems — all with a complete, queryable audit trail.

---

## Agent Roster

| Agent | File | Role |
|---|---|---|
| Orchestrator | `agents/orchestrator.agent.md` | Root coordinator; manages workflow state, retries, approvals |
| Planner | `agents/planner.agent.md` | LLM-based task decomposition using Foundry OSS model |
| Retriever | `agents/retriever.agent.md` | RAG retrieval via Azure AI Search |
| Policy / Safety | `agents/policy.agent.md` | Content safety, PII redaction, RBAC validation |
| Executor | `agents/executor.agent.md` | Tool/API calls via APIM gateway |

---

## What This System CAN Do

- Accept natural-language requests from authenticated enterprise users.
- Decompose requests into multi-step plans using an OSS LLM.
- Retrieve grounded answers from enterprise knowledge stores with citations.
- Execute registered tool/API calls (ITSM, CRM, ERP, HRIS, messaging) through APIM.
- Pause workflows for human approval before sensitive actions.
- Log every step for audit and compliance.
- Filter and redact PII before it crosses trust boundaries.
- Enforce RBAC so users only access data they are authorized to see.

---

## What This System CANNOT Do

- Access external internet URLs not registered in the tools registry.
- Execute actions without a valid Entra ID token.
- Bypass the Policy / Safety Agent gate under any circumstance.
- Store or transmit secrets outside of Azure Key Vault.
- Operate across environments (dev / nonprod / prod) without environment-scoped identities.
- Self-modify its own guardrails or approval policies at runtime.
- Proceed with a sensitive action if the approval step is denied or timed out.

---

## Behavioral Constraints

| Constraint | Rule |
|---|---|
| Determinism | Planner temperature is 0.1 — plans must be consistent and schema-valid |
| Grounding | All factual responses must include citations from AI Search (no hallucination policy) |
| Human oversight | Sensitive actions (write/execute on LOB systems) always require human approval |
| Auditability | Every step is logged — no silent failures |
| Least privilege | No agent holds permissions beyond its defined scope |
| Fail-safe | On any guardrail BLOCK, run terminates and user receives rationale |

---

## Guardrails

| Guardrail | File |
|---|---|
| Content Safety | `guardrails/content-safety.guardrail.md` |
| Access Control | `guardrails/access-control.guardrail.md` |
| Rate Limiting | `guardrails/rate-limiting.guardrail.md` |
| Data Governance | `guardrails/data-governance.guardrail.md` |
| Secrets Management | `guardrails/secrets-management.guardrail.md` |

---

## Spec Index

| Category | Files |
|---|---|
| Architecture | `specs/architecture-overview.md` |
| Functional Requirements | `specs/functional-requirements.md` |
| Non-Functional Requirements | `specs/non-functional-requirements.md` |
| Security & Governance | `specs/security-governance.md` |
| Azure Services Mapping | `specs/azure-services-mapping.md` |
| Orchestration Flow | `specs/orchestration-flow.md` |
| Message Schemas | `specs/message-schemas.md` |
| Evaluation | `specs/evaluation-spec.md` |
| RAI Assessment | `specs/rai-assessment.md` |
| Failure Modes | `specs/failure-modes.md` |
| Memory & State | `specs/memory-state.md` |
| API Contract | `specs/api-contract.md` |

---

## Ownership & Escalation

| Role | Responsibility |
|---|---|
| Product Owner | FR sign-off, RAI sign-off, go/no-go for prod |
| Platform Engineering | Infrastructure, IaC, deployment pipelines |
| AI/ML Team | Model selection, evaluation, prompt engineering |
| Security Team | Guardrail reviews, Key Vault, access reviews |
| Operations | Monitoring runbooks, incident response, DR drills |
