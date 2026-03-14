# Non-Functional Requirements

## NFR-1 — Availability

| Metric | Target |
|---|---|
| Orchestration + API layers SLA | 99.9% (per Azure regional SLA aggregation) |
| Foundry model endpoint SLA | 99.9% |
| AI Search SLA | 99.9% |

**Controls**: Multi-instance deployments, health probes, auto-restart via Durable Functions.

---

## NFR-2 — Performance

| Metric | Target |
|---|---|
| P50 end-to-end (read-only flows) | < 10 seconds |
| P95 end-to-end (action flows, excl. human approvals) | < 30 seconds |
| Foundry model inference latency (P50) | < 3 seconds |

**Controls**: Connection pooling, caching, optimized prompt sizes, AI Search replica tuning.

---

## NFR-3 — Scalability

| Component | Strategy |
|---|---|
| Durable Functions / Logic Apps | Horizontal auto-scale based on queue depth |
| Azure AI Search | Replica scaling per QPS |
| APIM | Auto-scale tiers or Premium with capacity units |
| Cosmos DB | Auto-scale RU/s |

---

## NFR-4 — Security

- Private Endpoints for AI Search, Storage, Cosmos DB, Key Vault.
- APIM inbound policies enforce JWT validation, IP filtering.
- mTLS or OAuth2 for all backend communication.
- No public keys or secrets exposed.
- VNet integration for all compute (Functions, Logic Apps).

**Guardrail Specs**: [access-control.guardrail.md](../guardrails/access-control.guardrail.md), [secrets-management.guardrail.md](../guardrails/secrets-management.guardrail.md).

---

## NFR-5 — Privacy

- PII minimization: only collect what is purpose-bound.
- Retention policies: time-bound, auto-purge.
- Optional redaction via Safety Agent before storage or response.

**Guardrail Spec**: [content-safety.guardrail.md](../guardrails/content-safety.guardrail.md).

---

## NFR-6 — Compliance

- All resources tagged per organizational policy.
- Azure Policy assignments enforce allowed SKUs, locations, and configurations.
- Defender for Cloud recommendations closed within policy SLAs.

**Guardrail Spec**: [data-governance.guardrail.md](../guardrails/data-governance.guardrail.md).

---

## NFR-7 — Observability

| Requirement | Target |
|---|---|
| Action-step trace coverage | 100% |
| Alert on failure rate spike | Within 5 minutes |
| Alert on latency breach | Within 5 minutes |
| Cost dashboards | Per resource group |

**Mapped Components**: Azure Monitor, Application Insights, Log Analytics.

---

## NFR-8 — Disaster Recovery

| Metric | Target |
|---|---|
| RPO (configuration) | ≤ 24 hours |
| RTO (stateless compute) | Redeploy from IaC within 1 hour |

**Controls**:
- Daily configuration backups (APIM, AI Search indexes/skillsets).
- Infrastructure as Code (Bicep/Terraform) in source control.
- Stateless compute redeployable from pipelines.
- Quarterly restore-runbook validation.

---

## NFR-9 — Cost Management

- Budgets and alerts per resource group.
- Workload auto-scaling to avoid over-provisioning.
- Vector index sizing guidelines documented.
- Monthly cost reviews.
