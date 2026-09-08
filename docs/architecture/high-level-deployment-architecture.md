# Enterprise Mortgage Platform — High-Level Deployment Architecture

**Document ID:** PLAT-001-T04-A  
**Parent Story:** PLAT-001 — Define microservice architecture baseline  
**Sub-task:** PLAT-001-T04 — Create high-level deployment architecture  
**Status:** Proposed

## 1. Purpose

Define the high-level deployment topology for the Enterprise Mortgage Platform so that independent microservices, platform infrastructure, databases, messaging, security, observability, and external integrations have clear runtime boundaries.

This document intentionally stays at architecture level. It does not define every Kubernetes manifest, Helm chart, firewall rule, or production sizing parameter.

---

## 2. Deployment Principles

### DEPLOY-RULE-001 — Independent Deployment

Each microservice is independently:

- built;
- versioned;
- containerized;
- deployed;
- scaled;
- rolled back.

No service is deployed as a module of another service.

### DEPLOY-RULE-002 — Stateless Application Pods

Business-service containers should remain stateless wherever practical.

State is externalized to:

- Oracle;
- Kafka;
- Redis where explicitly justified;
- external object/document storage where applicable.

### DEPLOY-RULE-003 — Database Isolation

Each persistent microservice accesses only its own Oracle schema, as defined in PLAT-001-T02.

### DEPLOY-RULE-004 — Messaging Through Kafka

Asynchronous cross-service event propagation uses Kafka.

### DEPLOY-RULE-005 — Gateway at Platform Edge

External clients enter through the API gateway. Internal business services are not directly internet-exposed.

### DEPLOY-RULE-006 — Identity Is Externalized

Authentication is delegated to an OIDC/OAuth2 identity provider.

### DEPLOY-RULE-007 — Observability Is Platform-Level

Metrics, logs, and traces are collected centrally without transferring business ownership.

### DEPLOY-RULE-008 — Environment Separation

Development, test, staging, and production are separate runtime environments with environment-specific configuration and secrets.

---

# 3. Runtime Building Blocks

## 3.1 Edge Layer

### `mortgage-api-gateway`

Responsibilities:

- public API entry point;
- routing;
- token validation;
- coarse authorization enforcement;
- correlation ID propagation;
- rate limiting where required;
- request/response logging with PII controls.

The gateway must not own mortgage business state.

---

## 3.2 Business Service Layer

Independently deployable services:

- customer-service
- mortgage-application-service
- borrower-financial-service
- property-service
- document-service
- mortgage-product-service
- credit-service
- appraisal-service
- fraud-risk-service
- policy-service
- underwriting-service
- manual-review-service
- conditions-service
- pricing-service
- closing-service
- loan-booking-service
- funding-service
- servicing-service
- payment-service
- notification-service
- audit-service

Each service is deployed separately and has:

- its own artifact;
- its own container image;
- its own configuration;
- its own health endpoints;
- its own resource limits;
- its own deployment lifecycle.

---

## 3.3 Data Layer

Oracle provides isolated schemas per service.

Examples:

```text
customer-service             -> CUSTOMER_SCHEMA
mortgage-application-service -> APPLICATION_SCHEMA
underwriting-service         -> UNDERWRITING_SCHEMA
servicing-service            -> SERVICING_SCHEMA
payment-service              -> PAYMENT_SCHEMA
```

No cross-schema application access is allowed.

---

## 3.4 Messaging Layer

Kafka provides the asynchronous event backbone.

Typical event families:

```text
mortgage.customer.events.v1
mortgage.application.events.v1
mortgage.credit.events.v1
mortgage.appraisal.events.v1
mortgage.policy.events.v1
mortgage.underwriting.events.v1
mortgage.conditions.events.v1
mortgage.closing.events.v1
mortgage.loan.events.v1
mortgage.funding.events.v1
mortgage.servicing.events.v1
mortgage.payment.events.v1
```

Supporting components may include:

- Schema Registry;
- Kafka Connect where justified;
- dead-letter/error topics;
- monitoring tooling.

---

## 3.5 Cache Layer

Redis may be used only where a documented use case exists, such as:

- short-lived read caching;
- rate-limit counters;
- non-authoritative session/platform state;
- deduplication/short-lived coordination where appropriate.

Redis must not become a hidden shared domain database.

---

## 3.6 Identity and Security Layer

An OIDC/OAuth2 identity provider issues tokens.

Typical flow:

```text
User / Client
    |
    | authenticate
    v
Identity Provider
    |
    | JWT access token
    v
API Gateway
    |
    v
Business Service
```

Services validate authorization claims appropriate to their resource boundary.

Secrets are provided through the platform secret-management mechanism rather than embedded in container images.

---

## 3.7 Observability Layer

Each service exposes operational telemetry.

Metrics:
- Spring Boot Actuator;
- Prometheus-compatible metrics.

Tracing:
- OpenTelemetry;
- distributed correlation/trace IDs.

Logging:
- structured application logs;
- centralized log aggregation.

Visualization/alerting:
- Grafana and related alerting platform.

Business audit records remain distinct from operational logs and are owned by `audit-service`.

---

## 3.8 External Integrations

Typical external dependencies include:

- credit bureau/provider;
- appraisal provider;
- fraud/risk provider;
- payment/funding provider;
- email/SMS provider;
- document/object storage.

External contracts should be isolated behind the owning integration service.

For example:

```text
underwriting-service
    -> credit-service
    -> external credit provider
```

`underwriting-service` should not integrate directly with the credit bureau.

---

# 4. Kubernetes Deployment Model

The target production-style runtime is Kubernetes.

Each service normally has:

```text
Deployment
Service
ConfigMap
Secret reference
ServiceAccount
NetworkPolicy
PodDisruptionBudget where required
HorizontalPodAutoscaler where justified
```

Ingress/external traffic terminates at the gateway/ingress layer, not at every business service.

Example:

```text
Internet
   |
   v
Ingress / Load Balancer
   |
   v
mortgage-api-gateway
   |
   +--> customer-service
   +--> mortgage-application-service
   +--> conditions-service
   +--> payment-service
```

---

# 5. Namespace Strategy

A simple environment-oriented model is recommended initially:

```text
mortgage-dev
mortgage-test
mortgage-staging
mortgage-prod
```

Within a namespace, individual services remain independent Kubernetes workloads.

A larger enterprise deployment may later separate namespaces by platform/domain/security boundary, but that is not required for PLAT-001-T04.

---

# 6. Configuration Strategy

Configuration should be externalized.

Examples:

- service endpoints;
- Kafka brokers;
- topic names;
- schema-registry URL;
- feature flags;
- timeout values;
- non-secret environment configuration.

Secrets must not be stored in Git as plaintext.

Secrets include:

- Oracle credentials;
- client secrets;
- external-provider credentials;
- private keys;
- messaging credentials.

---

# 7. Image and Artifact Flow

Expected deployment lifecycle:

```text
Developer commit
    |
    v
Git repository
    |
    v
CI pipeline
    |
    +--> compile
    +--> unit test
    +--> static/security checks
    +--> package
    +--> container build
    |
    v
Artifact / Container Registry
    |
    v
Deployment pipeline
    |
    v
Kubernetes environment
```

Each service has its own pipeline and release version.

---

# 8. Availability Model

At high level:

- run multiple replicas for production-critical stateless services;
- use readiness and liveness probes;
- avoid single-instance assumptions;
- scale services independently;
- deploy with rolling update or safer release strategy where justified;
- keep persistence and messaging as resilient managed/platform capabilities.

Exact replica counts and sizing are deferred to capacity/performance work.

---

# 9. Network Boundaries

Expected communication paths:

Allowed:

```text
Client -> Gateway
Gateway -> Business Service
Business Service -> Owning Business Service API
Business Service -> Kafka
Business Service -> Own Oracle Schema
Business Service -> Approved External Provider
Business Service -> Observability Platform
```

Forbidden:

```text
Internet -> Internal Business Service
Service A -> Service B Oracle Schema
Service A -> Service B internal database port
Service -> arbitrary external provider outside ownership boundary
```

---

# 10. Failure Domains

The deployment architecture must assume failure of:

- individual pods;
- individual service instances;
- network calls;
- external providers;
- Kafka consumers;
- transient Oracle connections.

Service design must not assume all dependencies are continuously available.

Detailed retry, timeout, circuit-breaker, DLQ, bulkhead, and disaster-recovery behavior is refined under later resilience/production-readiness epics.

---

# 11. Local Development Topology

Local development should mimic production boundaries without requiring production infrastructure.

A separate infrastructure repository may provide:

```text
mortgage-platform-local-infra/
├── docker-compose.yml
├── oracle/
├── kafka/
├── redis/
├── observability/
└── identity/
```

Individual microservice repositories remain separate.

Developers run only the services necessary for the workflow they are testing.

---

# 12. Deployment Ownership Boundaries

| Runtime Component | Ownership |
|---|---|
| Business-service container | Owning service team |
| Service configuration | Owning service team |
| Service Flyway migrations | Owning service team |
| Kubernetes platform | Platform/DevOps |
| Kafka platform | Platform/Eventing |
| Oracle platform | DBA/Platform |
| Identity platform | Security/IAM |
| Observability platform | Platform/SRE |
| Business audit data | audit-service |
| External provider adapter | Owning integration service |

---

# 13. Architecture Constraints

1. No monolithic deployment containing multiple business services.
2. No shared business database.
3. No direct internet exposure for internal services.
4. No service may depend on local filesystem persistence.
5. No hardcoded production credentials.
6. No service may require another service to be deployed in the same pod.
7. No shared business-domain runtime JAR as a coupling mechanism.
8. No distributed transaction manager spanning service databases.

---

# 14. Review Checklist

- [ ] Every business service has an independent deployment boundary.
- [ ] Gateway is the external entry point.
- [ ] Internal services are not directly internet-exposed.
- [ ] Oracle ownership matches PLAT-001-T02.
- [ ] Kafka is represented as the async backbone.
- [ ] OIDC/OAuth2 identity is represented.
- [ ] Observability components are included.
- [ ] External providers are isolated behind owning services.
- [ ] Environment separation is documented.
- [ ] Local-development topology preserves service independence.
