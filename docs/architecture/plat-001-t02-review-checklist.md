# PLAT-001-T02 — Review Checklist

## Ownership

- [ ] Every persistent business service has one assigned Oracle schema.
- [ ] Every schema has exactly one owning service.
- [ ] Schema ownership aligns with PLAT-001-T01 bounded contexts.
- [ ] Platform components are not treated as owners of mortgage business state.

## Isolation

- [ ] Cross-schema SELECT is prohibited.
- [ ] Cross-schema INSERT/UPDATE/DELETE/MERGE/DDL are prohibited.
- [ ] Cross-service foreign keys are prohibited.
- [ ] Database links are prohibited for service integration.
- [ ] Cross-schema synonyms are prohibited.
- [ ] Shared business tables are prohibited.

## Migration and Security

- [ ] Each service owns its Flyway migrations.
- [ ] One service's migration cannot modify another service's schema.
- [ ] Application DB credentials are intended to be service-specific.
- [ ] Cross-schema application grants are not part of the normal design.
- [ ] Least privilege is an explicit expectation.

## Integration Alternatives

- [ ] REST is defined for synchronous authoritative lookup.
- [ ] Kafka is defined for asynchronous propagation.
- [ ] Local projections are explicitly non-authoritative.
- [ ] Immutable historical snapshots are allowed for reproducibility.
- [ ] External IDs may be stored without cross-schema FKs.
- [ ] Distributed ACID transactions are prohibited.

## Mortgage Scenarios

- [ ] Application-to-customer access follows a service contract.
- [ ] Underwriting-to-credit access follows a service contract.
- [ ] Underwriting-to-policy access follows a service contract.
- [ ] Payment-to-servicing updates do not use direct SQL.

## Review Result

**Status:** Proposed / Approved / Approved with Actions / Rejected

**Reviewers**
- Tech Lead / Architect:
- Backend Engineer:
- DBA:
- Security:
- Mortgage SME / BA:

**Review date:**

**Actions/comments**
1.
2.
3.

## Definition of Done

PLAT-001-T02 may move to Done only when:

- both architecture documents are committed;
- this checklist is reviewed;
- unresolved ownership/access issues are captured as Jira follow-ups or ADRs;
- no unresolved database-boundary conflict blocks downstream implementation.
