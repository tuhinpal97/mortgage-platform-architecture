# Enterprise Mortgage Platform — REST Communication Matrix

**Document ID:** PLAT-001-T03-A  
**Parent Story:** PLAT-001 — Define microservice architecture baseline  
**Sub-task:** PLAT-001-T03 — Document REST and Kafka communication matrix  
**Status:** Proposed

## 1. Purpose

Define the initial synchronous service-to-service communication model for the Enterprise Mortgage Platform.

REST is used when a caller requires an immediate authoritative response and the interaction is naturally request/response.

This document defines:

- caller and target service;
- interaction purpose;
- synchronous dependency;
- expected API responsibility;
- retry/timeout expectations;
- ownership constraints;
- interactions that must not use direct database access.

---

## 2. REST Principles

### REST-RULE-001 — REST for Immediate Authoritative Response

Use REST when the caller must obtain a current answer before continuing.

Examples:

- validate customer;
- fetch current borrower financial facts;
- fetch current credit result;
- fetch current appraisal result;
- fetch applicable policy version;
- fetch product details.

### REST-RULE-002 — Owning Service Is the Authority

The target service owns the requested business data.

REST must not become a thin wrapper around another service's database.

### REST-RULE-003 — No Cascading Deep Call Chains

Avoid long synchronous chains such as:

```text
gateway -> application -> underwriting -> credit -> provider
```

for user-facing operations where practical.

Prefer orchestration, asynchronous progression, cached read models, or pre-computed facts where appropriate.

### REST-RULE-004 — Timeouts Are Mandatory

Every synchronous outbound call must have:

- connect timeout;
- read/response timeout;
- bounded retry policy;
- circuit breaker where appropriate;
- correlation/trace propagation.

### REST-RULE-005 — No Blind Retries for Non-Idempotent Operations

Retries are safe only when the operation is idempotent or protected by idempotency keys.

### REST-RULE-006 — Versioned Contracts

Public and service-to-service APIs should use versioned contracts such as:

`/api/v1/...`

### REST-RULE-007 — PII Minimization

Only the minimum required PII should be returned to the caller.

---

## 3. REST Communication Matrix

| Caller | Target | Purpose | Typical Interaction | Sync Required? | Notes |
|---|---|---|---|---|---|
| external-client | mortgage-api-gateway | Enter platform | HTTPS request | Yes | Gateway owns no business state |
| mortgage-api-gateway | customer-service | Customer operations | REST proxy/routing | Yes | AuthN/AuthZ enforced |
| mortgage-api-gateway | mortgage-application-service | Application operations | REST proxy/routing | Yes | No direct DB access |
| mortgage-api-gateway | borrower-financial-service | Borrower financial operations | REST proxy/routing | Yes | Scope-limited |
| mortgage-api-gateway | property-service | Property operations | REST proxy/routing | Yes | |
| mortgage-api-gateway | document-service | Document metadata/access | REST proxy/routing | Yes | Large binary handling may use storage-specific pattern |
| mortgage-api-gateway | mortgage-product-service | Product lookup | REST proxy/routing | Yes | |
| mortgage-api-gateway | conditions-service | Condition operations | REST proxy/routing | Yes | |
| mortgage-api-gateway | pricing-service | Pricing operations | REST proxy/routing | Yes | |
| mortgage-api-gateway | servicing-service | Servicing operations | REST proxy/routing | Yes | |
| mortgage-api-gateway | payment-service | Payment operations | REST proxy/routing | Yes | Idempotency required |
| mortgage-application-service | customer-service | Validate/read customer | GET customer summary/status | Yes | customer-service remains source of truth |
| borrower-financial-service | customer-service | Validate borrower/customer reference | GET customer existence/status | Usually yes | Avoid copying profile data |
| property-service | mortgage-application-service | Validate application reference | GET application summary | Usually yes | Could later use event-derived projection |
| document-service | mortgage-application-service | Validate application/business reference | GET application summary | Usually yes | Depends on document workflow |
| credit-service | customer-service | Get minimum identity facts required for bureau request | GET approved credit-input view | Yes | Strict PII minimization |
| appraisal-service | property-service | Retrieve property facts for appraisal | GET property summary | Yes | appraisal-service owns valuation result |
| fraud-risk-service | mortgage-application-service | Retrieve application context | GET risk-input view | Yes | Prefer purpose-built view |
| fraud-risk-service | customer-service | Retrieve permitted customer risk facts | GET risk-input view | Yes | PII controlled |
| underwriting-service | mortgage-application-service | Retrieve application facts | GET underwriting application view | Yes | Read-only contract |
| underwriting-service | borrower-financial-service | Retrieve verified financial facts | GET underwriting financial view | Yes | |
| underwriting-service | property-service | Retrieve property facts | GET underwriting property view | Yes | |
| underwriting-service | credit-service | Retrieve normalized credit result | GET credit result | Yes | |
| underwriting-service | appraisal-service | Retrieve appraisal result | GET appraisal result | Yes | |
| underwriting-service | fraud-risk-service | Retrieve fraud/risk result | GET risk result | Yes | |
| underwriting-service | policy-service | Resolve applicable policy version | GET applicable policy/version | Yes | Returned version should be immutable |
| pricing-service | mortgage-product-service | Retrieve product definition | GET product/version | Yes | |
| pricing-service | underwriting-service | Retrieve eligible underwriting outcome where required | GET decision summary | Yes | Avoid bidirectional cyclic dependency |
| manual-review-service | underwriting-service | Retrieve automated decision snapshot | GET evaluation summary | Yes | Human decision remains separate |
| manual-review-service | document-service | Retrieve document metadata/evidence | GET document metadata/access reference | Yes | |
| conditions-service | document-service | Retrieve evidence metadata | GET document metadata | Yes | Condition owns satisfaction state |
| closing-service | conditions-service | Verify condition completion | GET condition summary | Yes | |
| closing-service | pricing-service | Retrieve final approved pricing | GET pricing offer/version | Yes | |
| closing-service | mortgage-application-service | Retrieve application/borrower summary | GET closing view | Yes | |
| loan-booking-service | closing-service | Retrieve completed closing outcome | GET closing completion | Yes | Booking must be idempotent |
| funding-service | loan-booking-service | Retrieve booked loan terms | GET loan summary | Yes | |
| servicing-service | loan-booking-service | Retrieve booked terms if not fully event-carried | GET loan summary | Sometimes | Prefer event bootstrap where practical |
| payment-service | servicing-service | Validate servicing account/payment context | GET payment context | Yes | Payment must not update servicing DB |
| notification-service | customer-service | Resolve permitted destination/preferences if not event-carried | GET notification contact view | Sometimes | Avoid unnecessary PII |
| audit-service | source services | Ad hoc investigation only | GET resource by ID | Rare | Primary audit ingestion should be event-driven |

---

## 4. REST Interactions to Avoid

The following patterns should be treated as design smells unless explicitly approved:

```text
customer-service -> mortgage-application-service -> borrower-financial-service -> credit-service
```

during a single interactive request.

Avoid:

- chatty APIs;
- cyclic synchronous dependencies;
- generic "data service" APIs;
- APIs exposing raw database tables;
- service calls for data already safely carried in the initiating event;
- synchronous fan-out to many services on every request.

---

## 5. Example API Contracts

### Customer validation

```http
GET /api/v1/customers/{customerId}/summary
```

Possible response:

```json
{
  "customerId": "CUST-12345",
  "status": "ACTIVE",
  "kycStatus": "VERIFIED"
}
```

### Underwriting financial view

```http
GET /api/v1/borrowers/{customerId}/underwriting-financials?applicationId=APP-123
```

### Applicable policy

```http
GET /api/v1/policies/applicable?productCode=FIXED30&asOf=2026-09-08
```

The response must identify the immutable policy version used.

---

## 6. REST Review Checklist

- [ ] Every synchronous dependency has a clear business purpose.
- [ ] The target service is the authoritative owner of the requested data.
- [ ] No REST interaction exists merely to bypass schema ownership.
- [ ] Cyclic synchronous dependencies are avoided.
- [ ] Timeout/retry/idempotency expectations are documented.
- [ ] PII exposure is minimized.
- [ ] Versioned API contracts are expected.
- [ ] Deep synchronous call chains are avoided where practical.
