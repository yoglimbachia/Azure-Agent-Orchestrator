# Skill: Safety Filtering

## Identity

| Field | Value |
|---|---|
| Name | `safety-filtering` |
| Used By | Policy / Safety Agent |
| Backend | Azure AI Content Safety + custom logic |

## Description

Provides content safety checks, PII detection and redaction, secret-leakage detection, and business-rule enforcement. This skill is the enforcement mechanism for all guardrail specs.

## Capabilities

| Capability | Description |
|---|---|
| Content Safety | Toxicity, hate, violence, self-harm detection |
| PII Detection | Identify names, emails, phone numbers, national IDs, financial data |
| PII Redaction | Mask or remove PII before data crosses trust boundaries |
| Secret Detection | Pattern-match for API keys, connection strings, tokens in prompts |
| Business Rules | Evaluate org-specific policy rules (limits, scopes, geo) |
| RBAC Check | Validate user claims against required permissions per step |

## Content Safety Categories

| Category | Severity Levels | Action on HIGH |
|---|---|---|
| Hate | Low / Medium / High | BLOCK |
| Violence | Low / Medium / High | BLOCK |
| Self-Harm | Low / Medium / High | BLOCK |
| Sexual | Low / Medium / High | BLOCK |
| Jailbreak attempt | Detected / Not-detected | BLOCK |

## PII Entity Types

| Entity | Example | Redaction |
|---|---|---|
| PersonName | "John Smith" | `[PERSON]` |
| Email | "john@example.com" | `[EMAIL]` |
| Phone | "+44 7700 900000" | `[PHONE]` |
| NationalId | "AB 12 34 56 C" | `[NATIONAL_ID]` |
| CreditCard | "4111 1111 1111 1111" | `[CARD]` |
| IBAN | "GB29 NWBK 6016 1331 9268 19" | `[IBAN]` |

## Invocation — Content Safety

```
POST https://{content_safety_endpoint}/contentsafety/text:analyze?api-version=2024-09-01
Headers:
  Ocp-Apim-Subscription-Key: {key_from_keyvault}
  Content-Type: application/json
Body:
  {
    "text": "{input_text}",
    "categories": ["Hate", "Violence", "SelfHarm", "Sexual"],
    "outputType": "FourSeverityLevels"
  }
```

## Verdict Logic

```
IF any category severity == HIGH → BLOCK
IF secret_pattern_match → BLOCK
IF user lacks required RBAC role → BLOCK
IF business_rule violated → BLOCK
IF PII detected → ALLOW_WITH_CONDITIONS (redact and continue)
ELSE → ALLOW
```

## Output Schema

```json
{
  "verdict": "BLOCK",
  "rationale": "Content flagged for hate speech (severity: HIGH)",
  "categories": {
    "hate": "high",
    "violence": "low",
    "selfHarm": "none",
    "sexual": "none"
  },
  "piiDetected": [],
  "redactedText": null
}
```

## Guardrail Specs Enforced

- [content-safety.guardrail.md](../guardrails/content-safety.guardrail.md)
- [access-control.guardrail.md](../guardrails/access-control.guardrail.md)
- [secrets-management.guardrail.md](../guardrails/secrets-management.guardrail.md)

## Dependencies

- Azure AI Content Safety
- Azure Key Vault (subscription key)
- Microsoft Entra ID (user claims for RBAC checks)
