# Deployment Runbook

Step-by-step guide for bootstrapping and promoting the Azure Agentic Orchestrator across environments.

---

## Overview

| Environment | Trigger | Approver |
|---|---|---|
| Dev | Developer push to feature branch | Self (developer) |
| Non-Prod | PR merged to `main` | Peer review (1 approver) |
| Prod | Release tag + quality gate pass | Product Owner + Security sign-off |

---

## Pre-flight Checklist

Complete before **every** deployment.

- [ ] All evaluation thresholds pass (`specs/evaluation-spec.md` quality gates)
- [ ] No open P1 or P2 incidents in the target environment
- [ ] IaC (Bicep/Terraform) changes reviewed and tested in lower environment first
- [ ] `config/agent.config.yaml` values reviewed for target environment
- [ ] Key Vault secrets pre-created and verified accessible by Managed Identity
- [ ] `tools/tools-registry.yaml` reviewed — no unregistered tools added
- [ ] RAI assessment reviewed (`specs/rai-assessment.md`) if model or data has changed
- [ ] For Prod: ServiceNow change record created and approved ("SNOW on ADO")

---

## Environment Bootstrap Order

Deploy resources in this order to satisfy dependencies:

```
1. Resource Group
2. Managed Identity (User-Assigned)
3. Virtual Network + Subnets
4. Azure Key Vault (+ RBAC for Managed Identity)
   └── Populate required secrets
5. Azure Storage Account (+ Private Endpoint)
6. Azure Cosmos DB (+ Private Endpoint)
7. Azure AI Foundry / Model Deployment
8. Azure AI Search (+ Private Endpoint + index creation)
9. Azure App Insights + Log Analytics Workspace
10. Azure API Management (+ VNet integration)
    └── Import APIs from tools-registry.yaml
    └── Configure Key Vault named values
11. Azure Logic Apps / Durable Functions
    └── Assign Managed Identity
    └── Configure APIM endpoint
12. Azure Content Safety resource
13. Front-end UI (App Service / Static Web App)
```

---

## Step-by-Step: Non-Prod Deployment

### 1. Authenticate

```bash
az login
az account set --subscription "<subscription-id>"
```

### 2. Deploy Infrastructure (IaC)

```bash
cd infra/
az deployment group create \
  --resource-group rg-agent-nonprod \
  --template-file main.bicep \
  --parameters environment=nonprod
```

### 3. Populate Key Vault Secrets

For each secret in `config/agent.config.yaml` → `security.key_vault`:

```bash
az keyvault secret set \
  --vault-name "<keyvault-name>" \
  --name "kv-foundry-endpoint-key" \
  --value "<value>"
```

### 4. Create AI Search Index

```bash
curl -X PUT \
  "https://<search-service>.search.windows.net/indexes/enterprise-knowledge?api-version=2024-07-01" \
  -H "Authorization: Bearer $(az account get-access-token --resource https://search.azure.com --query accessToken -o tsv)" \
  -d @infra/search/index-definition.json
```

### 5. Deploy APIM APIs

```bash
az apim api import \
  --resource-group rg-agent-nonprod \
  --service-name "<apim-name>" \
  --api-id itsm-api \
  --specification-format OpenApi \
  --specification-path tools/openapi/itsm.yaml
```

### 6. Deploy Functions / Logic Apps

```bash
func azure functionapp publish "<function-app-name>"
```

### 7. Smoke Test

```bash
# Submit a test run
curl -X POST \
  "https://apim-agent-nonprod.azure-api.net/orchestrator/runs" \
  -H "Authorization: Bearer <test-token>" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "What is the server patching policy?"}'
```

Expected: `202 Accepted` with a `runId`.

---

## Rollback Procedure

If a deployment fails or causes P1/P2 incidents:

1. **Revert IaC**: re-deploy the previous Bicep/Terraform tag from source control.
2. **Revert function app**: `func azure functionapp publish` with previous release tag.
3. **Revert APIM**: restore APIM from the pre-deployment backup in Storage.
4. **Revert AI Search index**: re-apply previous index definition JSON.
5. **Notify**: post incident summary in ops channel; create post-mortem ticket.

---

## Prod Promotion Gate

In addition to pre-flight checklist, **prod** requires:

- [ ] Human review sign-off recorded in Azure DevOps (see `specs/evaluation-spec.md`)
- [ ] Security Team sign-off on guardrail changes (if any)
- [ ] RAI assessment re-confirmed (if model, data, or tool changes)
- [ ] ServiceNow change record in `Approved` state
- [ ] Backup taken of current APIM config: `az apim backup ...`
- [ ] Deployment window confirmed with operations team
