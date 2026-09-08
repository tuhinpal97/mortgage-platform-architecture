# PLAT-001-T01 Deliverable Package

This package contains the complete deliverables for **PLAT-001-T01 — Document bounded contexts and service responsibility matrix**.

## Files

```text
docs/
└── architecture/
    ├── bounded-contexts.md
    ├── service-responsibility-matrix.md
    └── plat-001-t01-review-checklist.md
```

## Recommended Repository

Create a dedicated architecture repository named `mortgage-platform-architecture`.

```text
mortgage-platform-architecture/
├── README.md
├── docs/
│   └── architecture/
│       ├── bounded-contexts.md
│       ├── service-responsibility-matrix.md
│       └── plat-001-t01-review-checklist.md
└── adr/
```

## Jira Execution

1. Move PLAT-001-T01 from READY to IN PROGRESS.
2. Add these documents to version control.
3. Review them using the checklist.
4. Resolve or track review comments.
5. Attach/link the repository commit or pull request in Jira.
6. Move through CODE REVIEW if required by the workflow.
7. Mark DONE only after review criteria are satisfied.

## Suggested Jira Completion Comment

```text
Completed PLAT-001-T01.

Deliverables:
- docs/architecture/bounded-contexts.md
- docs/architecture/service-responsibility-matrix.md
- docs/architecture/plat-001-t01-review-checklist.md

The documents define bounded contexts, authoritative business-data ownership,
Oracle schema boundaries, prohibited cross-service database access, external
reference rules, and initial service interaction responsibilities.

Architecture review completed and review actions have either been resolved or
captured as follow-up Jira work.
```

## Next Tasks

- PLAT-001-T02 — Document database ownership and no-cross-schema-access rule
- PLAT-001-T03 — Document REST and Kafka communication matrix
- PLAT-001-T04 — Create high-level deployment architecture
- PLAT-001-T05 — Review architecture against independent deployability principle

# PLAT-001-T02 Deliverable Package

This package contains the architecture deliverables for:

**PLAT-001-T02 — Define database ownership and no-cross-schema-access rules**

## Files

```text
docs/architecture/
├── database-ownership.md
├── database-access-rules.md
└── plat-001-t02-review-checklist.md
```

## Jira Acceptance Criteria

1. Every persistent microservice has an assigned Oracle schema.
2. Service-to-schema ownership is documented.
3. Direct cross-schema reads and writes are prohibited.
4. Cross-service foreign keys are prohibited.
5. Database links/synonyms cannot bypass service boundaries.
6. Cross-service data access uses REST, Kafka, approved projections, or immutable snapshots.
7. Each service owns its Flyway migrations.
8. A migration changes only its owning schema.
9. External identifiers may be stored without DB-level FK coupling.
10. Distributed ACID transactions across services are prohibited.
11. Concrete allowed/forbidden mortgage examples are documented.
12. Architecture review is completed before Done.

## Suggested Repository Location

```text
mortgage-platform-architecture/
└── docs/architecture/
```

## Suggested Jira Completion Comment

Use only after review is actually complete:

```text
Completed PLAT-001-T02.

Deliverables:
- database-ownership.md
- database-access-rules.md
- plat-001-t02-review-checklist.md

The architecture defines service-owned Oracle schemas, no-cross-schema-access
rules, service-owned Flyway migrations, prohibited cross-service FKs, and
approved REST/Kafka/projection/snapshot alternatives.

Review comments are resolved or tracked as explicit follow-up work.
```

