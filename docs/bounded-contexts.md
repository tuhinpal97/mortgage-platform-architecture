# Enterprise Mortgage Platform — Bounded Contexts

## 1. Document Control

**Document ID:** PLAT-001-T01-A  
**Parent Story:** PLAT-001 — Define microservice architecture baseline  
**Purpose:** Define the business bounded contexts for the Enterprise Mortgage Platform and establish conceptual ownership boundaries that guide independent microservice design.

## 2. Scope

This document defines the mortgage-domain bounded contexts that will be implemented as independently deployable services.

It establishes:

- the purpose of each bounded context;
- the business capabilities owned by each context;
- the principal aggregates and business facts owned by each context;
- responsibilities explicitly excluded from each context;
- the distinction between domain bounded contexts and platform capabilities;
- the architectural rules that prevent data and responsibility overlap.

This document does not define every API, Kafka topic, database table, or implementation class. Those are refined in later architecture and service-level design tasks.

## 3. Architecture Principles

### 3.1 Independent Deployability

Each business service is designed to be independently developed, versioned, built, tested, containerized, deployed, scaled, monitored, and released. No business service is implemented as a Maven module of another business service.

### 3.2 Single Authoritative Owner

A business fact has one authoritative bounded context. Other services may reference its identifier, consume it through a published API, consume events about it, or retain a historical snapshot where justified. Those actions do not transfer ownership.

### 3.3 Database Ownership

Each service owns its own Oracle schema/database boundary. Direct cross-service reads and writes, cross-service foreign keys, distributed joins across service schemas, and shared JPA entities are prohibited.

### 3.4 Communication

Use REST when an immediate synchronous response is required. Use Kafka when a business state change must be propagated asynchronously, multiple consumers are interested in the same business event, eventual consistency is acceptable, or temporal decoupling is beneficial.

### 3.5 Domain vs Platform Capability

The following are platform capabilities, not mortgage-domain bounded contexts: API Gateway, Identity/OIDC provider, Kafka, Schema Registry, Oracle platform, Redis, Prometheus, Grafana, OpenTelemetry, CI/CD, and Kubernetes.

# 4. Bounded Context Inventory

## 4.1 Customer & KYC Context

**Service:** `customer-service`  
**Primary purpose:** Maintain the authoritative customer identity, profile, contact information, addresses, and KYC status.

**Owns:** Customer, Customer Address, Contact Information, KYC Status, Customer Profile Status.

**Responsibilities:** Create and maintain customer profiles, manage addresses, track KYC status, expose customer data to authorized consumers, and protect customer PII.

**Does Not Own:** Mortgage application, borrower employment, income, assets, liabilities, credit bureau report, property, underwriting decision, loan servicing.

**Database Boundary:** `CUSTOMER_SCHEMA`

**Typical APIs:** `POST /api/v1/customers`, `GET /api/v1/customers/{customerId}`, `PATCH /api/v1/customers/{customerId}`, `GET /api/v1/customers/{customerId}/kyc`

**Candidate Events:** `CUSTOMER_CREATED`, `CUSTOMER_UPDATED`, `CUSTOMER_KYC_STATUS_CHANGED`

## 4.2 Mortgage Application Context

**Service:** `mortgage-application-service`  
**Primary purpose:** Manage the mortgage application lifecycle from draft through submission and later origination-state transitions.

**Owns:** Mortgage Application, Application Borrower Reference, Application Status, Application Status History, Application Submission State, Submission Idempotency Record.

**Responsibilities:** Create and retrieve applications, update drafts, associate borrowers, validate lifecycle transitions, submit applications, and publish submission-related events.

**Does Not Own:** Customer master data, borrower financial profile, property master data, credit report, underwriting policy, underwriting decision.

**Database Boundary:** `APPLICATION_SCHEMA`

**Important Rule:** The service may store `customerId` as an external reference, but it does not own the customer record and must not create a database foreign key into `CUSTOMER_SCHEMA`.

**Candidate Events:** `APPLICATION_CREATED`, `APPLICATION_SUBMITTED`, `APPLICATION_STATUS_CHANGED`

## 4.3 Borrower Financial Context

**Service:** `borrower-financial-service`  
**Primary purpose:** Maintain borrower financial facts used to evaluate repayment capacity and affordability.

**Owns:** Employment, Income, Asset, Liability, Financial Verification Status, Borrower Financial Summary.

**Responsibilities:** Maintain employment, declared and verified income, assets, liabilities, verification status, and expose normalized financial facts to underwriting.

**Does Not Own:** Customer profile, credit bureau report, underwriting decision, mortgage pricing, servicing balance.

**Database Boundary:** `FINANCIAL_SCHEMA`

## 4.4 Property Context

**Service:** `property-service`  
**Primary purpose:** Maintain authoritative facts about the subject property associated with a mortgage application.

**Owns:** Property, Property Address, Property Type, Property Ownership Information, Property Review Status.

**Does Not Own:** Professional appraisal, appraisal provider response, valuation review, mortgage application.

**Database Boundary:** `PROPERTY_SCHEMA`

**Boundary Clarification:** `property-service` owns property facts; `appraisal-service` owns appraisal workflow and appraisal results.

## 4.5 Document Context

**Service:** `document-service`  
**Primary purpose:** Manage mortgage document metadata, storage references, controlled access, and verification state.

**Owns:** Document Metadata, Document Storage Reference, Document Classification, Document Verification Status.

**Does Not Own:** The business fact represented by a document. A payslip may provide evidence of income, but verified income remains owned by `borrower-financial-service`.

**Database Boundary:** `DOCUMENT_SCHEMA`

## 4.6 Mortgage Product Context

**Service:** `mortgage-product-service`  
**Primary purpose:** Maintain the catalog and versions of mortgage products offered by the platform.

**Owns:** Mortgage Product, Product Version, Product Availability, Product Characteristics.

**Does Not Own:** Underwriting policies, underwriting rules, pricing decision, customer-specific underwriting decision.

**Database Boundary:** `PRODUCT_SCHEMA`

**Boundary Clarification:** Product definition describes what the lender offers. Underwriting policy describes how risk is evaluated. Pricing describes what terms/rate are offered for a specific case.

## 4.7 Credit Context

**Service:** `credit-service`  
**Primary purpose:** Integrate with external credit providers and expose normalized credit facts to the mortgage platform.

**Owns:** Credit Request, Provider Interaction Metadata, Normalized Credit Report, Credit Score, Retrieval Status.

**Does Not Own:** Customer profile, KYC, underwriting decision, borrower-declared liabilities.

**Database Boundary:** `CREDIT_SCHEMA`

## 4.8 Appraisal Context

**Service:** `appraisal-service`  
**Primary purpose:** Manage appraisal requests, provider integration, results, and appraisal review status.

**Owns:** Appraisal Request, Provider Interaction, Appraised Value, Appraisal Result, Appraisal Review Status.

**Does Not Own:** Property master data, mortgage application, underwriting decision.

**Database Boundary:** `APPRAISAL_SCHEMA`

## 4.9 Fraud / Risk Context

**Service:** `fraud-risk-service`  
**Primary purpose:** Evaluate fraud/risk indicators and expose normalized risk facts to underwriting.

**Owns:** Fraud Assessment, Risk Indicator, Fraud Assessment Result, External Risk Provider Interaction.

**Does Not Own:** Underwriting policy, underwriting final decision, customer profile, credit report.

**Database Boundary:** `RISK_SCHEMA`

## 4.10 Policy Management Context

**Service:** `policy-service`  
**Primary purpose:** Define, version, approve, activate, retire, and retrieve underwriting policies and rules.

**Owns:** Policy, Policy Version, Rule Definition, Rule Parameters, Policy Approval State, Activation Window, Policy Change Strategy.

**Responsibilities:** Create drafts, create immutable versions, configure rules, support maker-checker approval, schedule activation, retire previous versions without mutation, resolve applicable policy version, and preserve history.

**Does Not Own:** Mortgage application, underwriting evaluation, underwriting decision, manual review case.

**Database Boundary:** `POLICY_SCHEMA`

## 4.11 Automated Underwriting Context

**Service:** `underwriting-service`  
**Primary purpose:** Evaluate mortgage application facts against an applicable policy version and produce an automated underwriting outcome.

**Owns:** Underwriting Evaluation, Evaluation Snapshot, Rule Result, Calculated DTI/LTV, Automated Underwriting Decision, Policy Version Reference Used.

**Responsibilities:** Gather facts, resolve policy, calculate metrics, evaluate rules, resolve PASS/CONDITION/REFER/DECLINE, preserve policy version used, publish decision event.

**Does Not Own:** Policy authoring, customer master data, income source records, appraisal source record, credit provider integration, final human override.

**Database Boundary:** `UNDERWRITING_SCHEMA`

## 4.12 Manual Underwriting Context

**Service:** `manual-review-service`  
**Primary purpose:** Manage cases requiring human underwriter review.

**Owns:** Manual Review Case, Review Task, Assignment, Underwriter Recommendation, Manual Final Decision, Decision Rationale.

**Does Not Own:** Automated rule engine, policy definition, customer master data.

**Database Boundary:** `REVIEW_SCHEMA`

## 4.13 Conditions Context

**Service:** `conditions-service`  
**Primary purpose:** Manage underwriting/approval conditions and their lifecycle.

**Owns:** Condition, Condition Type, Condition Status, Evidence Reference, Satisfaction/Waiver Decision.

**Does Not Own:** Document binary content, underwriting policy, closing case.

**Database Boundary:** `CONDITION_SCHEMA`

## 4.14 Pricing Context

**Service:** `pricing-service`  
**Primary purpose:** Produce mortgage pricing offers based on product, case attributes, and pricing configuration.

**Owns:** Pricing Request, Pricing Result, Pricing Component, Pricing Assumption, Offer Version.

**Does Not Own:** Product master, underwriting decision, servicing calculations.

**Database Boundary:** `PRICING_SCHEMA`

## 4.15 Closing Context

**Service:** `closing-service`  
**Primary purpose:** Manage clear-to-close checks and closing workflow.

**Owns:** Closing Case, Closing Status, Closing Checklist, Closing Document References, Closing Completion.

**Does Not Own:** Booked loan, funding transaction, servicing account.

**Database Boundary:** `CLOSING_SCHEMA`

## 4.16 Loan Booking Context

**Service:** `loan-booking-service`  
**Primary purpose:** Convert a successfully closed mortgage into an authoritative booked loan.

**Owns:** Loan, Booked Loan Terms, Loan Booking Idempotency, Loan Booking Status.

**Does Not Own:** Funding transaction, payment transaction, servicing schedule.

**Database Boundary:** `LOAN_SCHEMA`

## 4.17 Funding Context

**Service:** `funding-service`  
**Primary purpose:** Manage funding/disbursement after a mortgage loan has been booked and is eligible for funding.

**Owns:** Funding Request, Funding Transaction, Funding Status, Funding Reconciliation State, Funding Idempotency.

**Does Not Own:** Loan terms, servicing account, payment posting.

**Database Boundary:** `FUNDING_SCHEMA`

## 4.18 Servicing Context

**Service:** `servicing-service`  
**Primary purpose:** Manage an active mortgage loan after funding.

**Owns:** Servicing Account, Payment Schedule, Principal Balance, Interest Balance, Delinquency State, Payoff State, Loan Closure State.

**Does Not Own:** Origination application, external payment transaction itself, underwriting decision.

**Database Boundary:** `SERVICING_SCHEMA`

## 4.19 Payments Context

**Service:** `payment-service`  
**Primary purpose:** Receive, validate, record, and process mortgage payment transactions.

**Owns:** Payment Transaction, Payment Idempotency, Payment Processing Status, Payment Reconciliation State.

**Does Not Own:** Authoritative loan balance, payment schedule, servicing account lifecycle.

**Database Boundary:** `PAYMENT_SCHEMA`

**Boundary Clarification:** `payment-service` owns the transaction; `servicing-service` owns the resulting loan balance.

## 4.20 Notification Context

**Service:** `notification-service`  
**Primary purpose:** Deliver business notifications triggered by mortgage platform events.

**Owns:** Notification Request, Notification Delivery Status, Notification Preference, Delivery Attempt.

**Does Not Own:** Source business state, customer profile, underwriting decision, payment posting.

**Database Boundary:** `NOTIFICATION_SCHEMA`

## 4.21 Business Audit Context

**Service:** `audit-service`  
**Primary purpose:** Maintain an immutable, searchable business audit trail across important mortgage events.

**Owns:** Audit Entry, Audit Event Projection, Correlation/Causation Trace.

**Does Not Own:** Operational application logs, source business state, source service records.

**Database Boundary:** `AUDIT_SCHEMA`

# 5. Context Relationships

- `customer-service` is upstream of `mortgage-application-service` for authoritative customer data.
- `borrower-financial-service`, `property-service`, `credit-service`, `appraisal-service`, `fraud-risk-service`, and `policy-service` are upstream fact/policy providers to `underwriting-service`.
- `underwriting-service` is upstream of `manual-review-service`, `conditions-service`, and later origination stages.
- `closing-service` is upstream of `loan-booking-service`.
- `loan-booking-service` is upstream of `funding-service`.
- `funding-service` is upstream of `servicing-service`.
- `payment-service` provides payment transaction outcomes to `servicing-service`.
- `audit-service` and `notification-service` are downstream event consumers for selected business events.

# 6. Ownership Rules

**BC-RULE-001 — One Authoritative Owner:** Every authoritative business fact has exactly one owning bounded context.

**BC-RULE-002 — No Cross-Service Database Access:** A service never reads or writes another service's database/schema directly.

**BC-RULE-003 — References Are Not Ownership:** A service may store another context's identifier as an external reference.

**BC-RULE-004 — Historical Snapshots Are Allowed:** A service may retain a local immutable snapshot when required for auditability, historical accuracy, decision reproducibility, or resilience. The snapshot does not become authoritative.

**BC-RULE-005 — Integration Uses Contracts:** Cross-context interaction occurs through versioned REST APIs and versioned Kafka events.

**BC-RULE-006 — Business Logic Stays With the Owner:** Business rules that mutate authoritative state belong to the owning context.

**BC-RULE-007 — No Shared Domain Model:** Business entities, repositories, and business services must not be shared as common source/JAR dependencies between services.

# 7. Example Ownership Decisions

| Question | Decision |
|---|---|
| Who owns customer name/address? | `customer-service` |
| Who owns borrower income? | `borrower-financial-service` |
| Who owns credit score? | `credit-service` |
| Who owns subject property? | `property-service` |
| Who owns appraised value? | `appraisal-service` |
| Who owns underwriting policy? | `policy-service` |
| Who owns automated underwriting result? | `underwriting-service` |
| Who owns human final review decision? | `manual-review-service` |
| Who owns booked loan terms? | `loan-booking-service` |
| Who owns funding transaction? | `funding-service` |
| Who owns active loan balance? | `servicing-service` |
| Who owns payment transaction? | `payment-service` |

# 8. Review Checklist

This document is considered reviewed when every planned business service appears in the inventory, each context has a clear purpose, authoritative ownership is unambiguous, database ownership is clear, domain contexts are distinguished from platform capabilities, and the design remains consistent with independent deployability.

# 9. Status

**Initial status:** Proposed  
**Expected approval:** Architecture review under PLAT-001  
**Change control:** Material boundary changes should be reviewed and recorded through an ADR where appropriate.
