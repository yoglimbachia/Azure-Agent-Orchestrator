# Security, Privacy & Governance Plan

## 1. Identity & Access

| Control | Details |
|---|---|
| User identity | Microsoft Entra ID for all human users |
| Service identity | Managed Identity for Functions, Logic Apps, APIM |
| Admin access | Conditional Access policies, MFA enforced |
| Least privilege | Minimal RBAC roles; no Owner on runtime identities |
| Access reviews | Periodic reviews via Entra Access Reviews |

**Baseline**: Group Azure standards for identity & least privilege.

---

## 2. Secrets, Keys & Certificates

| Control | Details |
|---|---|
| Storage | All secrets/certs in Azure Key Vault |
| APIM integration | Named values reference Key Vault |
| Rotation | Automatic rotation where supported |
| Code policy | Zero secrets in code, config, or pipeline variables |
| Crypto standards | No insecure algorithms; certificate validity per policy |

**Baseline**: APIM ↔ Key Vault integration standards.

---

## 3. Network Security

| Control | Details |
|---|---|
| Private Endpoints | AI Search, Storage, Cosmos DB, Key Vault |
| VNet integration | Functions, Logic Apps |
| Egress | Via Azure Firewall; deny direct internet where possible |
| Public access | Denied on data services by default |
| Enforcement | Azure Policy + Defender for Cloud |

---

## 4. API Governance

| Control | Details |
|---|---|
| Gateway | All tools exposed via APIM |
| Authentication | JWT validation (Entra ID tokens) |
| Policies | Rate limits, header scrubbing, request/response masking |
| Quotas | Per-product and per-user |
| Audit | Logs exported to Log Analytics |

**Baseline**: APIM security baseline.

---

## 5. Workflow & Integration Security

| Control | Details |
|---|---|
| Logic Apps | Run history secured; managed connectors only |
| Connections | Identity-based (no embedded creds) |
| Approval flows | Secured links; time-bound tokens |

**Baseline**: Logic Apps baseline standards.

---

## 6. Data Governance & Lineage

| Control | Details |
|---|---|
| Catalog | Datasets, prompts, derived artifacts registered in Microsoft Purview |
| Access | Via Entra Access Packages |
| Change management | Azure DevOps + ServiceNow ("SNOW on ADO") before prod |
| Lineage | Visible for at least one end-to-end workflow |

---

## 7. Responsible AI & Safety

| Control | Details |
|---|---|
| Content filtering | Toxicity and PII checks via Safety Agent |
| Tool-scoped permissions | Each tool has explicit allow-list per agent role |
| Human-in-the-loop | Mandatory for sensitive actions (FR-5) |
| Audit trail | Prompts, tool calls, parameter hashes, results logged |

---

## 8. Backup & Recovery

| Control | Details |
|---|---|
| APIM | Native backup to Storage account |
| AI Search | Index config exported as IaC |
| Infrastructure | Bicep/Terraform in source control |
| Validation | Quarterly restore-runbook drill |
