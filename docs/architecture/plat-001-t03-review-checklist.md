# PLAT-001-T03 — Communication Architecture Review Checklist

## Deliverables

- `rest-communication-matrix.md`
- `kafka-communication-matrix.md`
- `rest-vs-kafka-decision-guide.md`

## A. REST Review

- [X] Every synchronous dependency has a clear reason.
- [X] Target service is authoritative owner of requested data.
- [X] Cross-service DB access is not used.
- [X] Deep synchronous chains are avoided.
- [X] Cyclic REST dependencies are avoided.
- [X] Timeout and retry expectations are stated.
- [X] Non-idempotent retries require idempotency protection.
- [X] API versioning is expected.
- [X] PII minimization is explicit.

## B. Kafka Review

- [X] Every major event has one owning producer.
- [X] Events describe committed facts.
- [X] Major consumers are identified.
- [X] Transactional outbox is required.
- [X] Consumers are idempotent.
- [X] At-least-once delivery is assumed.
- [X] Event metadata includes correlation/causation.
- [X] Event versioning is required.
- [X] PII minimization is explicit.
- [X] DLQ/error handling is acknowledged.

## C. Architecture Consistency

- [X] REST/Kafka choices are consistent with bounded contexts from PLAT-001-T01.
- [X] Communication choices preserve DB isolation from PLAT-001-T02.
- [X] No service depends on another service's database.
- [X] Audit and notification are primarily event-driven.
- [X] State-change producer owns the event.
- [X] No obvious synchronous dependency cycle remains.

## D. Review Result

**Status:** Proposed / Approved / Approved with Actions / Rejected

**Reviewers**
- Tech Lead / Architect: Tuhin Pal
- Backend Engineer: Tuhin Pal
- Platform/Kafka Engineer: Tuhin Pal
- Security: Tuhin Pal
- Mortgage SME / BA: Tuhin Pal

**Review date:** 08/09/2026

**Actions/comments**
1.
2.
3.

## Definition of Done

PLAT-001-T03 may move to Done when:

- all communication matrices are committed;
- REST and Kafka ownership rules are reviewed;
- no unresolved critical dependency cycle remains;
- event producers/consumers are clear enough for downstream implementation;
- open design questions are captured as Jira follow-up work or ADRs.
