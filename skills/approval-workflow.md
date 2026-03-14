# Skill: Approval Workflow

## Identity

| Field | Value |
|---|---|
| Name | `approval-workflow` |
| Used By | Orchestrator Agent |
| Backend | Azure Logic Apps / Durable Functions + Teams / Email connectors |

## Description

Manages human-in-the-loop approval workflows for sensitive actions. Pauses orchestration, sends approval requests via messaging channels, and resumes or aborts based on the decision.

## Capabilities

| Capability | Description |
|---|---|
| Approval Request | Send a structured approval request to designated approvers |
| Multi-Channel Notification | Teams adaptive card and/or email with approve/deny links |
| Timeout Handling | Auto-deny or escalate if no response within SLA |
| Decision Capture | Record approve/deny with approver identity and timestamp |
| Workflow Resume | Unblock the paused orchestration on decision |

## Approval Flow

```
Orchestrator detects requiresApproval == true
    │
    ▼
┌──────────────────────────────┐
│  Build approval request      │
│  - Action summary            │
│  - Risk level                │
│  - Requester identity        │
│  - Correlation ID            │
└──────────┬───────────────────┘
           │
           ▼
┌──────────────────────────────┐
│  Send notification           │
│  - Teams adaptive card       │
│  - Email with secure link    │
└──────────┬───────────────────┘
           │
           ▼
┌──────────────────────────────┐
│  Wait for decision           │
│  - Approve / Deny            │
│  - Timeout: 4 hours default  │
└──────────┬───────────────────┘
           │
     ┌─────┴──────┐
     │             │
  APPROVE        DENY / TIMEOUT
     │             │
     ▼             ▼
  Resume run    Abort run
  (continue      (log: denied
   to Executor)   or timed_out)
```

## Approval Request Schema

```json
{
  "correlationId": "abc-123",
  "requester": "user@org.com",
  "action": "Create ITSM ticket for server provisioning",
  "riskLevel": "medium",
  "plannedTool": "itsm.createTicket",
  "parameters": { /* non-PII summary */ },
  "approvers": ["manager@org.com"],
  "timeoutMinutes": 240
}
```

## Configuration

| Parameter | Default | Notes |
|---|---|---|
| Approval timeout | 240 minutes (4 hours) | Configurable per policy |
| Escalation on timeout | Auto-deny | Can be changed to escalate |
| Notification channels | Teams + Email | At least one required |
| Min approvers | 1 | Multi-approver supported |

## Dependencies

- Azure Logic Apps (managed connectors for Teams / Email)
- Microsoft Entra ID (approver identity resolution)
- Azure Monitor (log approval decisions)
