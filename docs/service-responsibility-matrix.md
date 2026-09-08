# Enterprise Mortgage Platform — Service Responsibility Matrix

## 1. Document Control

**Document ID:** PLAT-001-T01-B  
**Parent Story:** PLAT-001 — Define microservice architecture baseline  
**Purpose:** Provide an implementation-oriented reference for service ownership, data authority, persistence boundaries, and prohibited responsibilities.

# 2. Service Responsibility Matrix

| Service | Primary Responsibility | Authoritative Business Data | Oracle Schema | Explicitly Does Not Own |
|---|---|---|---|---|
| `mortgage-api-gateway` | External API entry point, routing, coarse access enforcement | No mortgage business state | None | Customer, application, underwriting, loan state |
| `customer-service` | Customer profile and KYC | Customer, address, contact data, KYC status | `CUSTOMER_SCHEMA` | Application, income, credit, underwriting |
| `mortgage-application-service` | Mortgage application lifecycle | Application, borrower references, status history | `APPLICATION_SCHEMA` | Customer master, finances, underwriting |
| `borrower-financial-service` | Borrower financial profile | Employment, income, assets, liabilities, verification status | `FINANCIAL_SCHEMA` | Credit report, underwriting decision |
| `property-service` | Subject property facts | Property, ownership, property review state | `PROPERTY_SCHEMA` | Appraisal |
| `document-service` | Document metadata/access/verification | Document metadata, storage reference, verification state | `DOCUMENT_SCHEMA` | Source business fact represented by document |
| `mortgage-product-service` | Mortgage product catalog | Product, product version, product characteristics | `PRODUCT_SCHEMA` | Underwriting policy, case pricing |
| `credit-service` | Credit provider integration | Credit request/report/score | `CREDIT_SCHEMA` | Customer master, underwriting decision |
| `appraisal-service` | Property appraisal | Appraisal request/result/value | `APPRAISAL_SCHEMA` | Property master |
| `fraud-risk-service` | Fraud/risk assessment | Fraud assessment, indicators, result | `RISK_SCHEMA` | Underwriting policy/final decision |
| `policy-service` | Underwriting policy management | Policy, version, rule definition, approvals | `POLICY_SCHEMA` | Application evaluation/decision |
| `underwriting-service` | Automated underwriting | Evaluation, derived metrics, rule results, automated decision | `UNDERWRITING_SCHEMA` | Policy authoring, source customer/credit/property data |
| `manual-review-service` | Human underwriting | Review case, assignment, human decision, rationale | `REVIEW_SCHEMA` | Automated policy engine |
| `conditions-service` | Underwriting/approval conditions | Condition, status, evidence reference, waiver/satisfaction | `CONDITION_SCHEMA` | Document binary, policy definition |
| `pricing-service` | Mortgage case pricing | Pricing request/result/components/assumptions | `PRICING_SCHEMA` | Product master, underwriting decision |
| `closing-service` | Closing workflow | Closing case/status/checklist | `CLOSING_SCHEMA` | Booked loan, servicing |
| `loan-booking-service` | Loan creation after closing | Loan, booked terms, booking status | `LOAN_SCHEMA` | Funding transaction, payment processing |
| `funding-service` | Funding/disbursement | Funding request/transaction/status/reconciliation | `FUNDING_SCHEMA` | Servicing balance |
| `servicing-service` | Active loan servicing | Servicing account, schedule, balances, delinquency/payoff state | `SERVICING_SCHEMA` | Origination application, payment transaction itself |
| `payment-service` | Payment transaction processing | Payment transaction, payment status, idempotency/reconciliation | `PAYMENT_SCHEMA` | Authoritative loan balance |
| `notification-service` | Business notifications | Notification request/preferences/delivery outcome | `NOTIFICATION_SCHEMA` | Source business state |
| `audit-service` | Immutable business audit trail | Audit entries/correlation trace | `AUDIT_SCHEMA` | Source business state, operational logs |

# 3. Authoritative Business Fact Matrix

| Business Fact | Authoritative Owner |
|---|---|
| Customer identity | `customer-service` |
| Customer address | `customer-service` |
| Customer KYC status | `customer-service` |
| Mortgage application | `mortgage-application-service` |
| Mortgage application status | `mortgage-application-service` |
| Application-to-borrower relationship | `mortgage-application-service` |
| Borrower employment | `borrower-financial-service` |
| Borrower income | `borrower-financial-service` |
| Borrower assets | `borrower-financial-service` |
| Borrower liabilities | `borrower-financial-service` |
| Subject property | `property-service` |
| Property ownership | `property-service` |
| Document metadata | `document-service` |
| Document verification status | `document-service` |
| Mortgage product definition | `mortgage-product-service` |
| Credit report | `credit-service` |
| Credit score | `credit-service` |
| Appraised value | `appraisal-service` |
| Fraud/risk assessment | `fraud-risk-service` |
| Underwriting policy | `policy-service` |
| Underwriting policy version | `policy-service` |
| Automated underwriting evaluation | `underwriting-service` |
| Automated underwriting decision | `underwriting-service` |
| Manual review case | `manual-review-service` |
| Human underwriting decision | `manual-review-service` |
| Underwriting condition | `conditions-service` |
| Condition satisfaction/waiver | `conditions-service` |
| Pricing offer | `pricing-service` |
| Closing case | `closing-service` |
| Booked mortgage loan | `loan-booking-service` |
| Funding transaction | `funding-service` |
| Servicing account | `servicing-service` |
| Authoritative loan balance | `servicing-service` |
| Payment transaction | `payment-service` |
| Notification delivery state | `notification-service` |
| Business audit entry | `audit-service` |

# 4. External Reference Rules

| Consuming Service | External Reference | Owning Service | Rule |
|---|---|---|---|
| `mortgage-application-service` | `customerId` | `customer-service` | Store identifier only; no DB FK to customer schema |
| `borrower-financial-service` | `applicationId`, `customerId` | application/customer contexts | External references only |
| `property-service` | `applicationId` | `mortgage-application-service` | External reference only |
| `document-service` | `applicationId`, business object reference | owning context | No ownership transfer |
| `credit-service` | `applicationId`, `customerId` | application/customer contexts | Used for request correlation |
| `appraisal-service` | `applicationId`, `propertyId` | application/property contexts | No direct DB access |
| `underwriting-service` | application/fact/policy IDs | multiple contexts | Persist references and decision snapshots |
| `closing-service` | `applicationId` | application context | Closing does not become application owner |
| `loan-booking-service` | `closingId`, `applicationId` | closing/application contexts | Used to establish provenance |
| `funding-service` | `loanId` | loan-booking context | Funding owns transaction, not loan terms |
| `servicing-service` | `loanId` | loan-booking context | Creates its own servicing aggregate |
| `payment-service` | `servicingAccountId` / `loanId` | servicing/loan contexts | Payment result affects servicing through contract |

# 5. Allowed vs Forbidden Data Access

## Allowed

- REST calls to published service APIs;
- Kafka event consumption;
- local projections or snapshots created from contracts;
- storing another context's identifier as an external reference;
- local derived values when source and derivation are documented.

## Forbidden

- `SELECT` from another service's Oracle schema;
- `INSERT`, `UPDATE`, or `DELETE` against another service's schema;
- database links used to bypass service APIs;
- cross-service foreign keys;
- shared JPA entity modules;
- shared repositories across business services;
- direct dependency on another service's business implementation JAR.

# 6. Initial Service Interaction Matrix

| Caller / Producer | Target / Consumer | Mechanism | Purpose | Ownership Note |
|---|---|---|---|---|
| External Client | `mortgage-api-gateway` | HTTPS | Enter platform | Gateway owns no business state |
| `mortgage-api-gateway` | `customer-service` | REST | Customer operations | Customer data owned by customer-service |
| `mortgage-api-gateway` | `mortgage-application-service` | REST | Application operations | Application data owned by application-service |
| `mortgage-application-service` | `customer-service` | REST | Validate/read customer summary | No CUSTOMER_SCHEMA access |
| `mortgage-application-service` | Kafka | Event | `APPLICATION_SUBMITTED` | Application service owns state change |
| `underwriting-service` | `mortgage-application-service` | REST | Retrieve application facts | Consumes, does not own |
| `underwriting-service` | `borrower-financial-service` | REST | Retrieve financial facts | No FINANCIAL_SCHEMA access |
| `underwriting-service` | `property-service` | REST | Retrieve property facts | No PROPERTY_SCHEMA access |
| `underwriting-service` | `credit-service` | REST | Retrieve credit facts | No CREDIT_SCHEMA access |
| `underwriting-service` | `appraisal-service` | REST | Retrieve appraisal result | No APPRAISAL_SCHEMA access |
| `underwriting-service` | `fraud-risk-service` | REST | Retrieve risk facts | No RISK_SCHEMA access |
| `underwriting-service` | `policy-service` | REST | Retrieve applicable immutable policy version | Records exact version used |
| `underwriting-service` | Kafka | Event | Underwriting decision event | Decision owned by underwriting-service |
| `manual-review-service` | Kafka / REST | Event + REST | Receive referred case and fetch facts | Human decision separately owned |
| `conditions-service` | Kafka / REST | Event + REST | Create/manage conditions | Document evidence remains in document-service |
| `closing-service` | Kafka / REST | Event + REST | Progress eligible case to closing | Closing owns closing workflow |
| `loan-booking-service` | Kafka / REST | Event + REST | Book closed loan | Booking is idempotent |
| `loan-booking-service` | Kafka | Event | `LOAN_BOOKED` | Source of booked-loan event |
| `funding-service` | Kafka / REST | Event + REST | Fund booked loan | Funding owns funding transaction |
| `funding-service` | Kafka | Event | `LOAN_FUNDED` | Funding service owns state change |
| `servicing-service` | Kafka | Event | Create servicing account from funded loan | Creates local aggregate |
| `payment-service` | Kafka / REST | Event + REST | Process payment transaction | Payment owns transaction |
| `payment-service` | Kafka | Event | `PAYMENT_POSTED` | Servicing consumes result |
| `audit-service` | Kafka | Consumer | Build audit trail | Does not modify source state |
| `notification-service` | Kafka | Consumer | Trigger customer communications | Does not own source event state |

# 7. Boundary Rules

**BR-001 — Database Isolation:** A service must never directly read or write another business service's database/schema.

**BR-002 — No Cross-Service Foreign Keys:** A database foreign key may reference only tables owned by the same service/schema.

**BR-003 — External Identifiers:** Identifiers from other contexts may be stored as scalar external references.

**BR-004 — Historical Snapshot:** A local immutable snapshot may be retained for auditability, reproducibility, historical correctness, or resilience. The authoritative source remains the owner.

**BR-005 — Published Contracts:** Service integration must use published REST or Kafka contracts.

**BR-006 — No Shared Business JARs:** Business services may not share domain entities, repositories, business service implementations, or business aggregates.

**BR-007 — Local Transactions:** Each service owns its own ACID transaction boundary. Distributed database transactions across services are prohibited.

**BR-008 — Event Publication Ownership:** The service that commits an authoritative state change publishes the corresponding event where required.

**BR-009 — PII Minimization:** APIs, events, logs, traces, and audit records must expose only the minimum sensitive data required.

**BR-010 — Independent Migration Ownership:** Each service owns its Flyway migrations and database evolution.

# 8. Responsibility Conflict Examples

## Income
`borrower-financial-service` owns borrower income. `underwriting-service` may consume it and store an immutable decision snapshot, but does not become authoritative owner.

## Appraised Value
`property-service` owns property facts. `appraisal-service` owns appraised value and appraisal result.

## Policy Rule
`policy-service` owns immutable policy/rule versions. `underwriting-service` evaluates them and records which version was used.

## Payment vs Loan Balance
`payment-service` owns the payment transaction. `servicing-service` owns the authoritative loan balance and applies the payment result according to servicing rules.

# 9. PLAT-001-T01 Acceptance Mapping

The task-specific acceptance criteria are satisfied when:

1. all planned mortgage business contexts are listed;
2. every context has one clear primary responsibility;
3. authoritative business facts have a single owner;
4. database/schema ownership is documented;
5. important exclusions are documented;
6. direct cross-service database access is explicitly prohibited;
7. external-reference rules are defined;
8. service interaction boundaries are documented at a high level;
9. domain contexts are distinguished from platform capabilities;
10. the documents are reviewed for consistency with independent deployability.

# 10. Status

**Initial status:** Proposed  
**Review owner:** Tech Lead / Architect  
**Review participants:** Backend representative, DBA, Security representative, Product/Domain representative where ownership is unclear  
**Next related tasks:** PLAT-001-T02, PLAT-001-T03, PLAT-001-T04, PLAT-001-T05
