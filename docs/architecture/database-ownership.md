# Enterprise Mortgage Platform — Database Ownership

**Document ID:** PLAT-001-T02-A  
**Parent Story:** PLAT-001  
**Sub-task:** PLAT-001-T02 — Define database ownership and no-cross-schema-access rules  
**Status:** Proposed

## Purpose

Define the Oracle persistence ownership model for the Enterprise Mortgage Platform.

The core rule is:

> Each persistent microservice owns one logical Oracle schema boundary and is the only application service allowed to directly access that schema.

A schema is an internal implementation detail of its owning service, not a shared integration layer.

## Service-to-Schema Ownership Matrix

| Service | Oracle Schema | Authoritative Persistence |
|---|---|---|
| customer-service | CUSTOMER_SCHEMA | Customer, address, contact data, KYC |
| mortgage-application-service | APPLICATION_SCHEMA | Application, borrower references, application status/history |
| borrower-financial-service | FINANCIAL_SCHEMA | Employment, income, assets, liabilities |
| property-service | PROPERTY_SCHEMA | Property facts and ownership |
| document-service | DOCUMENT_SCHEMA | Document metadata, storage reference, verification status |
| mortgage-product-service | PRODUCT_SCHEMA | Product definitions and versions |
| credit-service | CREDIT_SCHEMA | Credit request, normalized report, score |
| appraisal-service | APPRAISAL_SCHEMA | Appraisal request/result/value |
| fraud-risk-service | RISK_SCHEMA | Fraud/risk assessment |
| policy-service | POLICY_SCHEMA | Policy, version, rules, approvals |
| underwriting-service | UNDERWRITING_SCHEMA | Evaluation, rule result, automated decision |
| manual-review-service | REVIEW_SCHEMA | Manual review case and human decision |
| conditions-service | CONDITION_SCHEMA | Conditions and satisfaction/waiver state |
| pricing-service | PRICING_SCHEMA | Pricing request/result/offer |
| closing-service | CLOSING_SCHEMA | Closing case/checklist/status |
| loan-booking-service | LOAN_SCHEMA | Booked loan and terms |
| funding-service | FUNDING_SCHEMA | Funding transaction/status |
| servicing-service | SERVICING_SCHEMA | Servicing account, schedule, balances |
| payment-service | PAYMENT_SCHEMA | Payment transaction/status |
| notification-service | NOTIFICATION_SCHEMA | Notification request/delivery state |
| audit-service | AUDIT_SCHEMA | Immutable business audit entries |

`mortgage-api-gateway` does not own mortgage business persistence.

## What Ownership Includes

The owning service is responsible for:

- tables, indexes, sequences/identity strategy;
- constraints within its own boundary;
- outbox and processed-event/idempotency tables;
- Flyway migrations;
- schema evolution;
- local retention and cleanup logic;
- operational documentation for its persistence.

## Migration Ownership

Each service keeps Flyway migrations in its own repository.

Example:

```text
customer-service/
└── src/main/resources/db/migration/
    ├── V1__create_customer.sql
    ├── V2__create_customer_address.sql
    └── V3__create_outbox_event.sql
```

A migration may modify only the schema owned by that service.

## External Identifiers

A service may store another service's identifier as a logical reference.

Example:

`mortgage-application-service` may store `customerId`.

It must not create an Oracle foreign key to `CUSTOMER_SCHEMA.CUSTOMER`.

## Historical Snapshots

A service may store an immutable snapshot of data owned elsewhere when needed for audit, reproducibility, historical correctness, or resilience.

Example: underwriting may store the exact income, credit score, appraisal value, and policy version used for a past decision. That snapshot does not become the current source of truth.

## Ownership Examples

### Customer vs Application

- `customer-service` owns customer data.
- `mortgage-application-service` owns the application.
- The application stores only `customerId` as an external reference.

### Payment vs Servicing

- `payment-service` owns payment transactions.
- `servicing-service` owns the authoritative loan balance.
- Payment must not update `SERVICING_SCHEMA` directly.

### Policy vs Underwriting

- `policy-service` owns policy versions.
- `underwriting-service` owns evaluations and decisions.
- Underwriting stores the policy version reference used, but does not query `POLICY_SCHEMA` directly.

## Review Checklist

- [ ] Every persistent business service has an assigned schema.
- [ ] Every schema has one owner.
- [ ] Migration ownership is clear.
- [ ] External IDs are logical references only.
- [ ] Cross-service foreign keys are prohibited.
- [ ] Historical snapshots do not create conflicting ownership.
