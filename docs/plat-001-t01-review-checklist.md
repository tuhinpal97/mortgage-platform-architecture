# PLAT-001-T01 — Architecture Review Checklist

## Deliverables Under Review

- `bounded-contexts.md`
- `service-responsibility-matrix.md`

## Domain Boundary Review

- [x] Every planned business service has a defined purpose.
- [x] Each bounded context is understandable without implementation code.
- [x] Ambiguous responsibilities are explicitly excluded.
- [x] Product, policy, pricing, underwriting, closing, funding, servicing, and payment responsibilities are clearly separated.

## Data Ownership Review

- [x] Every major business fact has one authoritative owner.
- [x] Each service's Oracle schema boundary is clear.
- [x] Cross-service foreign keys are prohibited.
- [x] Direct cross-service SQL access is prohibited.
- [x] External-reference rules are documented.

## Integration Review

- [x] Synchronous vs asynchronous interaction principles are stated.
- [x] REST and Kafka are the planned business integration mechanisms.
- [x] Event publication responsibility is tied to the service that owns the state change.
- [x] Local snapshots/projections are distinguished from authoritative data.

## Independent Deployability Review

- [x] Each business service can evolve without sharing domain source code.
- [x] Shared business JARs are prohibited.
- [x] Each service owns its database migrations.
- [x] The architecture avoids distributed ACID transactions across services.

## Security / Compliance Review

- [x] Customer PII ownership is explicit.
- [x] PII minimization rules are present.
- [x] Document access ownership is explicit.
- [x] Audit and operational logging responsibilities are separated.

## Review Result

**Status:** Proposed / Approved / Approved with Actions / Rejected

**Reviewer(s):**

- Tech Lead / Architect
- Backend Engineer:
- DBA:
- Security:
- Mortgage SME / BA:

**Review date:**

**Actions / comments:**

1.
2.
3.

## Definition of Done for PLAT-001-T01

The Jira sub-task may move to Done when both architecture documents exist in version control, this checklist has been completed, no unresolved ownership conflict blocks downstream design, review comments are incorporated or tracked as explicit follow-up work, and the parent PLAT-001 story remains consistent with these decisions.
