# Tools Registry
# Central registry of all tools callable by the Executor Agent via APIM.
# Each tool maps a logical name to an APIM API operation.

version: "1.0"

tools:

  # ── Line-of-Business Tools ────────────────────────────────────
  itsm.createTicket:
    description: Create a support ticket in the ITSM system
    apim_api: itsm-api
    method: POST
    path: /tickets
    auth: oauth2-managed-identity
    requiresApproval: true
    parameters:
      - name: title
        type: string
        required: true
      - name: description
        type: string
        required: true
      - name: priority
        type: string
        enum: [low, medium, high, critical]
        required: true

  itsm.getTicket:
    description: Retrieve a support ticket by ID
    apim_api: itsm-api
    method: GET
    path: /tickets/{ticketId}
    auth: oauth2-managed-identity
    requiresApproval: false

  crm.getContact:
    description: Retrieve a CRM contact record
    apim_api: crm-api
    method: GET
    path: /contacts/{contactId}
    auth: oauth2-managed-identity
    requiresApproval: false

  erp.getOrder:
    description: Retrieve an ERP order record
    apim_api: erp-api
    method: GET
    path: /orders/{orderId}
    auth: oauth2-managed-identity
    requiresApproval: false

  hris.getEmployee:
    description: Retrieve an HRIS employee record
    apim_api: hris-api
    method: GET
    path: /employees/{employeeId}
    auth: oauth2-managed-identity
    requiresApproval: false

  # ── Messaging / Notification Tools ────────────────────────────
  messaging.sendTeamsMessage:
    description: Send a message or adaptive card via Microsoft Teams
    apim_api: messaging-api
    method: POST
    path: /teams/messages
    auth: oauth2-managed-identity
    requiresApproval: false
    parameters:
      - name: channel
        type: string
        required: true
      - name: content
        type: object
        required: true

  messaging.sendEmail:
    description: Send an email notification
    apim_api: messaging-api
    method: POST
    path: /email/send
    auth: oauth2-managed-identity
    requiresApproval: false
    parameters:
      - name: to
        type: string
        required: true
      - name: subject
        type: string
        required: true
      - name: body
        type: string
        required: true

  # ── Internal API Tools ────────────────────────────────────────
  internal.queryApi:
    description: Generic call to an internal line-of-business API
    apim_api: internal-api
    method: "{dynamic}"
    path: "{dynamic}"
    auth: oauth2-managed-identity
    requiresApproval: true
    parameters:
      - name: apiName
        type: string
        required: true
      - name: operation
        type: string
        required: true
      - name: payload
        type: object
        required: false
