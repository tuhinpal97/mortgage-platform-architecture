# PLAT-001-T02 — Review Checklist

## Ownership

* \[X] Every persistent business service has one assigned Oracle schema.
* \[X] Every schema has exactly one owning service.
* \[X] Schema ownership aligns with PLAT-001-T01 bounded contexts.
* \[X] Platform components are not treated as owners of mortgage business state.

## Isolation

* \[X] Cross-schema SELECT is prohibited.
* \[X] Cross-schema INSERT/UPDATE/DELETE/MERGE/DDL are prohibited.
* \[X] Cross-service foreign keys are prohibited.
* \[X] Database links are prohibited for service integration.
* \[X] Cross-schema synonyms are prohibited.
* \[X] Shared business tables are prohibited.

## Migration and Security

* \[X] Each service owns its Flyway migrations.
* \[X] One service's migration cannot modify another service's schema.
* \[X] Application DB credentials are intended to be service-specific.
* \[X] Cross-schema application grants are not part of the normal design.
* \[X] Least privilege is an explicit expectation.

## Integration Alternatives

* \[X] REST is defined for synchronous authoritative lookup.
* \[X] Kafka is defined for asynchronous propagation.
* \[X] Local projections are explicitly non-authoritative.
* \[X] Immutable historical snapshots are allowed for reproducibility.
* \[X] External IDs may be stored without cross-schema FKs.
* \[X] Distributed ACID transactions are prohibited.

## Mortgage Scenarios

* \[X] Application-to-customer access follows a service contract.
* \[X] Underwriting-to-credit access follows a service contract.
* \[X] Underwriting-to-policy access follows a service contract.
* \[X] Payment-to-servicing updates do not use direct SQL.

## Review Result

**Status:** Proposed / Approved / Approved with Actions / Rejected

**Reviewers**

* Tech Lead / Architect: Tuhin Pal
* Backend Engineer: Tuhin Pal
* DBA: Tuhin Pal
* Security: Tuhin Pal
* Mortgage SME / BA: Tuhin Pal

**Review date: 08/09/2026**

**Actions/comments**
1.
2.
3.

## Definition of Done

PLAT-001-T02 may move to Done only when:

* both architecture documents are committed;
* this checklist is reviewed;
* unresolved ownership/access issues are captured as Jira follow-ups or ADRs;
* no unresolved database-boundary conflict blocks downstream implementation.

