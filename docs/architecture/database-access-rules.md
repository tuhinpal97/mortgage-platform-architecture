# Enterprise Mortgage Platform — Database Access Rules

**Document ID:** PLAT-001-T02-B  
**Status:** Proposed

## Fundamental Rule

> A business microservice may directly access only the Oracle schema that it owns.

## DB-RULE-001 — Own-Schema Access Only

Allowed:

```text
customer-service -> CUSTOMER_SCHEMA
underwriting-service -> UNDERWRITING_SCHEMA
payment-service -> PAYMENT_SCHEMA
```

Forbidden:

```text
underwriting-service -> CREDIT_SCHEMA
payment-service -> SERVICING_SCHEMA
mortgage-application-service -> CUSTOMER_SCHEMA
```

## DB-RULE-002 — Cross-Schema SELECT Is Prohibited

Forbidden:

```sql
SELECT score
FROM CREDIT_SCHEMA.CREDIT_REPORT
WHERE application_id = :applicationId;
```

Correct:

```text
underwriting-service -> REST -> credit-service -> CREDIT_SCHEMA
```

or consume an approved Kafka event/projection.

## DB-RULE-003 — Cross-Schema Writes Are Prohibited

Cross-schema `INSERT`, `UPDATE`, `DELETE`, `MERGE`, and DDL are prohibited.

Wrong:

```text
payment-service -> UPDATE SERVICING_SCHEMA.LOAN_ACCOUNT
```

Correct:

```text
payment-service
  -> PAYMENT_POSTED event
  -> Kafka
  -> servicing-service
  -> SERVICING_SCHEMA
```

## DB-RULE-004 — Cross-Service Foreign Keys Are Prohibited

Forbidden:

```sql
ALTER TABLE APPLICATION_SCHEMA.APPLICATION_BORROWER
ADD CONSTRAINT fk_customer
FOREIGN KEY (customer_id)
REFERENCES CUSTOMER_SCHEMA.CUSTOMER(id);
```

Store `customer_id` as an external scalar reference instead.

## DB-RULE-005 — Database Links Are Prohibited for Application Integration

Oracle database links must not be used to bypass service contracts.

## DB-RULE-006 — Cross-Schema Synonyms Are Prohibited

A synonym must not disguise access to another service's schema.

## DB-RULE-007 — Shared Business Tables Are Prohibited

Two services must never share write ownership of one business table.

## DB-RULE-008 — Published Contracts Are the Integration Boundary

Cross-service data access must use:

1. versioned REST;
2. versioned Kafka events;
3. approved local projection/read model;
4. approved immutable historical snapshot.

## DB-RULE-009 — Flyway Ownership Is Local

A service's migration may not alter another service's objects, create cross-service constraints, or seed another service's business data.

## DB-RULE-010 — Distributed ACID Transactions Are Prohibited

Use local ACID transactions plus:

- transactional outbox;
- idempotent consumers;
- eventual consistency;
- compensation where needed.

## DB-RULE-011 — Local Projections Do Not Transfer Ownership

A cached/event-derived projection remains non-authoritative unless architecture ownership is explicitly changed.

## DB-RULE-012 — Cross-Schema Grants Are Not an Integration Mechanism

Application users should not receive normal cross-schema SELECT/UPDATE grants.

Administrative identities for backup, monitoring, or DBA work are separate from application integration.

## DB-RULE-013 — Service-Specific DB Credentials

Production application credentials should be scoped to the owning schema and follow least privilege.

## DB-RULE-014 — No Cross-Service ORM Mapping

A JPA entity in one service must not map another service's table.

## DB-RULE-015 — Reporting Is Not an Exception

Operational/reporting needs should use approved projections, reporting services, or analytical stores—not direct transactional cross-schema joins.

## Allowed Patterns

| Need | Preferred Pattern |
|---|---|
| Immediate authoritative value | REST |
| Asynchronous state propagation | Kafka |
| Efficient local read with eventual consistency | Local projection |
| Exact historical decision reproduction | Immutable snapshot |
| Change another service's state | Owning service contract, never direct SQL |

## Concrete Mortgage Examples

### Application validates customer

Wrong:

```text
mortgage-application-service -> CUSTOMER_SCHEMA
```

Correct:

```text
mortgage-application-service -> customer-service REST API
```

### Underwriting reads credit

Wrong:

```text
underwriting-service -> CREDIT_SCHEMA
```

Correct:

```text
underwriting-service -> credit-service REST API
```

### Payment affects loan balance

Wrong:

```text
payment-service -> UPDATE SERVICING_SCHEMA
```

Correct:

```text
payment-service -> PAYMENT_POSTED -> Kafka -> servicing-service
```

### Underwriting uses policy

Wrong:

```text
underwriting-service -> POLICY_SCHEMA
```

Correct:

```text
underwriting-service -> policy-service API -> immutable policy version
```

## Enforcement Strategy

Future platform/security implementation should reinforce these rules with:

- separate Oracle users per service;
- no normal cross-schema application grants;
- service-specific secrets;
- schema-scoped Flyway users;
- CI checks for foreign-schema references in migrations;
- architecture/code review checks.

## Exceptions

Any exception requires documented justification, architecture review, security/operational impact analysis, and an ADR. No exception is approved by this document.
