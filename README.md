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
