# Enterprise Mortgage Platform — REST vs Kafka Decision Guide

**Document ID:** PLAT-001-T03-C  
**Status:** Proposed

## Purpose

Provide a practical decision rule for choosing synchronous REST versus asynchronous Kafka communication.

## Decision Table

| Question | Prefer REST | Prefer Kafka |
|---|---|---|
| Does caller need an immediate authoritative response? | Yes | No |
| Is the interaction naturally request/response? | Yes | No |
| Must multiple consumers react independently? | No | Yes |
| Can processing continue eventually? | No | Yes |
| Is temporal decoupling valuable? | Sometimes | Yes |
| Is user waiting on the result? | Often | Usually no |
| Is this a state-change notification? | Usually no | Yes |
| Does producer need to know all consumers? | Usually yes | No |
| Does consumer need historical replay? | Rarely | Often |
| Is failure handling expected via retries/DLQ? | Limited | Yes |

## Examples

### Use REST

```text
underwriting-service -> policy-service
```

Reason: underwriting needs the applicable policy version before completing the decision.

### Use Kafka

```text
mortgage-application-service
    -> APPLICATION_SUBMITTED
    -> underwriting-service
    -> credit-service
    -> appraisal-service
    -> fraud-risk-service
```

Reason: submission is a committed business fact with multiple downstream consumers.

### Hybrid

```text
APPLICATION_SUBMITTED
    -> underwriting-service

underwriting-service
    -> REST -> borrower-financial-service
    -> REST -> credit-service
    -> REST -> appraisal-service
```

The event starts the workflow; REST obtains authoritative facts required for a specific evaluation.

## Anti-Patterns

### REST for event broadcasting

Bad:

```text
application-service
  -> call credit-service
  -> call appraisal-service
  -> call fraud-service
  -> call notification-service
  -> call audit-service
```

Prefer publishing `APPLICATION_SUBMITTED` where consumers can react independently.

### Kafka for simple immediate lookup

Bad:

```text
underwriting-service
  -> publish REQUEST_CUSTOMER_DATA
  -> wait for CUSTOMER_DATA_RESPONSE
```

for a straightforward immediate lookup.

Prefer REST unless asynchronous decoupling is a deliberate architectural requirement.

## Rule of Thumb

> REST asks a service for something now. Kafka announces that something happened.
