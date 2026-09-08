# Enterprise Mortgage Platform — Kafka Communication Matrix

**Document ID:** PLAT-001-T03-B  
**Parent Story:** PLAT-001  
**Sub-task:** PLAT-001-T03 — Document REST and Kafka communication matrix  
**Status:** Proposed

## 1. Purpose

Define the initial asynchronous business-event communication model for the Enterprise Mortgage Platform.

Kafka is used to propagate business state changes asynchronously, decouple producers from consumers, and support eventual consistency.

---

## 2. Kafka Principles

### KAFKA-RULE-001 — Business Events Represent Facts

Events should describe something that already happened.

Examples:

- `APPLICATION_SUBMITTED`
- `CREDIT_REPORT_AVAILABLE`
- `UNDERWRITING_DECIDED`
- `LOAN_BOOKED`
- `LOAN_FUNDED`
- `PAYMENT_POSTED`

### KAFKA-RULE-002 — Producer Owns the State Change

The service that owns and commits the authoritative state change publishes the corresponding event.

### KAFKA-RULE-003 — Transactional Outbox

Business state change and event publication intent should be persisted in one local transaction using a transactional outbox pattern.

### KAFKA-RULE-004 — Idempotent Consumers

Consumers must tolerate duplicate delivery using a processed-event/idempotency mechanism.

### KAFKA-RULE-005 — No PII Dumping

Events must contain only business data necessary for consumers.

### KAFKA-RULE-006 — Versioned Event Contracts

Event schemas must be versioned and backward compatibility rules documented.

### KAFKA-RULE-007 — Correlation and Causation

Events should carry metadata such as:

- eventId;
- eventType;
- eventVersion;
- occurredAt;
- correlationId;
- causationId;
- producer;
- aggregateId.

---

## 3. Initial Kafka Communication Matrix

| Producer | Event | Primary Consumers | Purpose |
|---|---|---|---|
| customer-service | CUSTOMER_CREATED | audit-service, notification-service (if applicable) | Propagate customer creation fact |
| customer-service | CUSTOMER_UPDATED | audit-service, approved projections | Propagate customer profile changes |
| customer-service | CUSTOMER_KYC_STATUS_CHANGED | mortgage-application-service (if needed), audit-service | Propagate KYC lifecycle state |
| mortgage-application-service | APPLICATION_CREATED | audit-service | Record application creation |
| mortgage-application-service | APPLICATION_SUBMITTED | underwriting-service, credit-service, appraisal-service, fraud-risk-service, audit-service, notification-service | Start downstream origination processing |
| mortgage-application-service | APPLICATION_STATUS_CHANGED | audit-service, notification-service | Propagate lifecycle state |
| borrower-financial-service | BORROWER_FINANCIALS_UPDATED | underwriting-service (if reevaluation model requires), audit-service | Signal changed borrower facts |
| document-service | DOCUMENT_REGISTERED | audit-service | Record document metadata creation |
| document-service | DOCUMENT_VERIFIED | conditions-service, underwriting-service if required, audit-service | Signal verified evidence |
| credit-service | CREDIT_REPORT_REQUESTED | audit-service | Track request lifecycle |
| credit-service | CREDIT_REPORT_AVAILABLE | underwriting-service, audit-service | Signal normalized credit result availability |
| credit-service | CREDIT_REPORT_FAILED | underwriting-service/manual-review-service as policy requires, audit-service | Signal credit retrieval failure |
| appraisal-service | APPRAISAL_REQUESTED | audit-service | Track appraisal lifecycle |
| appraisal-service | APPRAISAL_COMPLETED | underwriting-service, audit-service | Signal appraisal result availability |
| appraisal-service | APPRAISAL_FAILED | underwriting-service/manual-review-service as required, audit-service | Signal failure |
| fraud-risk-service | FRAUD_RISK_ASSESSED | underwriting-service, audit-service | Signal risk facts availability |
| policy-service | POLICY_VERSION_APPROVED | audit-service | Record policy approval |
| policy-service | POLICY_VERSION_ACTIVATED | underwriting-service cache/projection if used, audit-service | Notify active policy change |
| policy-service | POLICY_VERSION_RETIRED | audit-service | Record retirement |
| underwriting-service | UNDERWRITING_STARTED | audit-service | Track decision workflow |
| underwriting-service | UNDERWRITING_DECIDED | manual-review-service, conditions-service, pricing-service, audit-service, notification-service | Propagate automated decision |
| manual-review-service | MANUAL_REVIEW_CREATED | audit-service, notification-service/internal workflow | Signal manual case |
| manual-review-service | MANUAL_REVIEW_DECIDED | conditions-service, pricing-service, audit-service, notification-service | Propagate human decision |
| conditions-service | CONDITION_CREATED | document-service/notification-service where appropriate, audit-service | Signal condition issuance |
| conditions-service | CONDITION_SATISFIED | closing-service, audit-service | Signal fulfillment |
| conditions-service | ALL_MANDATORY_CONDITIONS_SATISFIED | closing-service, audit-service | Enable closing progression |
| pricing-service | PRICING_COMPLETED | closing-service, audit-service | Signal approved pricing outcome |
| closing-service | CLEAR_TO_CLOSE | notification-service, audit-service | Signal readiness |
| closing-service | CLOSING_COMPLETED | loan-booking-service, audit-service | Trigger loan booking |
| loan-booking-service | LOAN_BOOKED | funding-service, servicing-service, audit-service | Publish booked loan fact |
| funding-service | FUNDING_INITIATED | audit-service | Track funding |
| funding-service | LOAN_FUNDED | servicing-service, notification-service, audit-service | Start post-funding lifecycle |
| funding-service | FUNDING_FAILED | notification-service/internal operations, audit-service | Signal failure/reconciliation need |
| servicing-service | SERVICING_ACCOUNT_CREATED | notification-service, audit-service | Signal servicing setup |
| servicing-service | PAYMENT_DUE | notification-service | Payment reminder trigger |
| payment-service | PAYMENT_RECEIVED | audit-service | Record inbound transaction |
| payment-service | PAYMENT_POSTED | servicing-service, notification-service, audit-service | Apply payment result downstream |
| payment-service | PAYMENT_FAILED | notification-service, audit-service | Signal failure |
| servicing-service | LOAN_PAID_OFF | notification-service, audit-service | Signal payoff |
| notification-service | NOTIFICATION_SENT | audit-service if business-required | Record delivery outcome |
| notification-service | NOTIFICATION_FAILED | audit-service/internal ops | Record delivery failure |

---

## 4. Topic Strategy

Initial logical topic families may follow domain-event groupings such as:

```text
mortgage.customer.events.v1
mortgage.application.events.v1
mortgage.document.events.v1
mortgage.credit.events.v1
mortgage.appraisal.events.v1
mortgage.risk.events.v1
mortgage.policy.events.v1
mortgage.underwriting.events.v1
mortgage.conditions.events.v1
mortgage.pricing.events.v1
mortgage.closing.events.v1
mortgage.loan.events.v1
mortgage.funding.events.v1
mortgage.servicing.events.v1
mortgage.payment.events.v1
mortgage.notification.events.v1
```

Exact topic partitioning, retention, schema registry configuration, and naming standards should be finalized under the Kafka/event architecture epic.

---

## 5. Event Envelope

Recommended baseline:

```json
{
  "eventId": "uuid",
  "eventType": "APPLICATION_SUBMITTED",
  "eventVersion": 1,
  "occurredAt": "2026-09-08T10:30:00Z",
  "producer": "mortgage-application-service",
  "aggregateType": "MortgageApplication",
  "aggregateId": "APP-12345",
  "correlationId": "uuid",
  "causationId": "uuid",
  "data": {}
}
```

---

## 6. Delivery Semantics

The architecture assumes at-least-once delivery.

Therefore:

- producers use transactional outbox;
- consumers are idempotent;
- duplicate events must not duplicate business effects;
- ordering assumptions must be explicit;
- retries must not silently discard events;
- poison messages require DLQ/error-handling strategy.

---

## 7. Kafka Interactions to Avoid

Avoid:

- commands disguised as facts without clear ownership;
- exposing entire database rows in events;
- embedding sensitive PII unnecessarily;
- one giant enterprise topic for all domains;
- assuming exactly-once business processing without idempotency;
- direct synchronous dependency hidden inside an event consumer;
- consumers mutating producer-owned state directly.

---

## 8. Kafka Review Checklist

- [ ] Every event has a clear owning producer.
- [ ] Events represent committed business facts.
- [ ] Consumers are listed for major lifecycle events.
- [ ] Transactional outbox is the producer baseline.
- [ ] Consumer idempotency is required.
- [ ] At-least-once delivery assumption is explicit.
- [ ] Correlation/causation metadata is defined.
- [ ] PII minimization is stated.
- [ ] Versioning/backward compatibility is required.
- [ ] DLQ/error-handling strategy is acknowledged.
