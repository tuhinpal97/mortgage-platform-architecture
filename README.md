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

# PLAT-001-T03 Deliverable Package

This package contains the complete deliverables for:

**PLAT-001-T03 — Document REST and Kafka communication matrix**

## Files

```text
docs/architecture/
├── rest-communication-matrix.md
├── kafka-communication-matrix.md
├── rest-vs-kafka-decision-guide.md
└── plat-001-t03-review-checklist.md
```

## What This Task Establishes

PLAT-001-T01 defined domain/service ownership.

PLAT-001-T02 defined database ownership and prohibited direct cross-schema access.

PLAT-001-T03 now defines how those isolated services communicate:

- REST for immediate authoritative request/response;
- Kafka for asynchronous business-event propagation.

## Recommended Jira Acceptance Criteria

1. Major synchronous service-to-service REST dependencies are documented.
2. Every REST dependency has a clear business purpose and authoritative target owner.
3. Deep/cyclic synchronous dependencies are explicitly discouraged.
4. Major mortgage lifecycle Kafka events are documented.
5. Each event has one owning producer and identified consumers.
6. Transactional outbox is defined as the producer baseline.
7. Consumer idempotency is required.
8. At-least-once delivery semantics are explicit.
9. Event versioning and correlation/causation metadata are defined.
10. PII minimization applies to REST and Kafka contracts.
11. A REST-vs-Kafka decision guide is documented.
12. Communication design is reviewed for consistency with PLAT-001-T01 and PLAT-001-T02.

## Suggested Jira Completion Comment

Use only after review:

```text
Completed PLAT-001-T03.

Deliverables:
- rest-communication-matrix.md
- kafka-communication-matrix.md
- rest-vs-kafka-decision-guide.md
- plat-001-t03-review-checklist.md

The platform now has an initial service communication model covering synchronous
REST dependencies, asynchronous Kafka business events, producer/consumer ownership,
transactional outbox, idempotent consumers, and REST-vs-Kafka selection rules.

Review comments are resolved or tracked as follow-up work.
```

# PLAT-001-T04 Deliverable Package

This package contains the complete deliverables for:

**PLAT-001-T04 — Create high-level deployment architecture**

## Files

```text
docs/architecture/
├── high-level-deployment-architecture.md
├── environment-topology.md
├── high-level-deployment-diagram.md
└── plat-001-t04-review-checklist.md
```

## What This Task Establishes

PLAT-001-T01 defined service/domain ownership.

PLAT-001-T02 defined database/schema ownership.

PLAT-001-T03 defined REST and Kafka communication.

PLAT-001-T04 defines where those components run and how they are separated at runtime.

## Recommended Jira Acceptance Criteria

1. Every microservice has an independent deployment boundary.
2. The API gateway is the external business-service entry point.
3. Internal services are not directly internet-exposed.
4. Kubernetes is documented as the target production-style runtime.
5. Oracle service-owned schema boundaries are represented.
6. Kafka is represented as the asynchronous event backbone.
7. OIDC/OAuth2 identity integration is represented.
8. Observability includes metrics, logs, and distributed tracing.
9. External providers are isolated behind their owning services.
10. DEV, TEST, STAGING, and PROD separation is documented.
11. CI -> registry -> deployment flow is documented.
12. Local development preserves independent service repositories.
13. A source-controlled high-level deployment diagram is produced.
14. Architecture is reviewed for consistency with PLAT-001-T01, T02, and T03.

## Suggested Jira Completion Comment

Use only after actual review:

```text
Completed PLAT-001-T04.

Deliverables:
- high-level-deployment-architecture.md
- environment-topology.md
- high-level-deployment-diagram.md
- plat-001-t04-review-checklist.md

The deployment baseline now defines independent Kubernetes workloads, gateway
entry, service-owned Oracle schemas, Kafka messaging, identity, observability,
external-provider boundaries, environment separation, and CI/CD deployment flow.

Review comments are resolved or tracked as follow-up work.
```



