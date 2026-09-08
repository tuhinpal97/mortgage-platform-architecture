# Enterprise Mortgage Platform — Environment and Deployment Topology

**Document ID:** PLAT-001-T04-B  
**Status:** Proposed

## 1. Environment Model

| Environment | Purpose | Typical Data | Deployment Style |
|---|---|---|---|
| DEV | Developer/team integration | Synthetic | Frequent deployment |
| TEST | Automated/integration testing | Synthetic/test fixtures | CI-driven |
| STAGING | Production-like validation | Masked/synthetic | Controlled release |
| PROD | Customer/business workload | Production | Governed release |

Each environment has isolated configuration, credentials, endpoints, and data.

---

## 2. Logical Production Topology

```text
                        +----------------------+
                        |   External Clients   |
                        +----------+-----------+
                                   |
                                   v
                        +----------------------+
                        | LB / Ingress / WAF   |
                        +----------+-----------+
                                   |
                                   v
                        +----------------------+
                        | mortgage-api-gateway |
                        +----------+-----------+
                                   |
                 +-----------------+------------------+
                 |                 |                  |
                 v                 v                  v
        +----------------+ +----------------+ +----------------+
        | customer       | | application    | | payment        |
        | service        | | service        | | service        |
        +-------+--------+ +-------+--------+ +-------+--------+
                |                  |                  |
                v                  v                  v
        CUSTOMER_SCHEMA    APPLICATION_SCHEMA   PAYMENT_SCHEMA

Internal service-to-service REST flows remain inside the platform network.

Business events flow through Kafka.

Every service emits telemetry to the observability platform.
```

---

## 3. Domain-Service Topology

```text
                            API GATEWAY
                                |
     +--------------------------+-----------------------------+
     |                          |                             |
     v                          v                             v
 Customer                 Application                    Servicing/Payment
     |                          |
     |                          +------------------------------+
     |                                                         |
     v                                                         v
Financial / Property / Document / Product              Origination Workflow
                                                               |
                    +----------------+-------------------------+----------------+
                    |                |                         |                |
                    v                v                         v                v
                 Credit          Appraisal                  Fraud/Risk       Policy
                    \                |                         /                /
                     \               |                        /                /
                      +---------------+-----------------------+----------------+
                                      |
                                      v
                               Underwriting
                                      |
                         +------------+-------------+
                         |                          |
                         v                          v
                  Manual Review                Conditions
                         \                          /
                          +------------+-----------+
                                       |
                                       v
                                    Pricing
                                       |
                                       v
                                    Closing
                                       |
                                       v
                                Loan Booking
                                       |
                                       v
                                    Funding
                                       |
                                       v
                                   Servicing
                                       |
                                       v
                                    Payment
```

This is a logical interaction view, not a runtime call sequence.

---

## 4. Platform Services

```text
+--------------------------------------------------------------+
|                         Kubernetes                            |
|                                                              |
|  [Gateway] [Business Services] [Workers/Consumers]           |
|                                                              |
+--------------------------------------------------------------+
       |             |              |              |
       v             v              v              v
    Oracle         Kafka          Redis        Observability
                                   |        Prometheus/Grafana
                                   |        OpenTelemetry/Logs
                                   |
                               only approved
                               cache use cases
```

---

## 5. External Provider Boundary

```text
credit-service      -> Credit Bureau
appraisal-service   -> Appraisal Provider
fraud-risk-service  -> Fraud/Risk Provider
funding-service     -> Funding/Payment Rail
payment-service     -> Payment Provider
notification-service-> Email/SMS Provider
document-service    -> Object/Document Storage
```

Business services that do not own the integration must not bypass these adapter services.

---

## 6. Security Zones

High-level logical zones:

```text
Public Zone
  -> Load Balancer / WAF / Ingress

Edge Zone
  -> mortgage-api-gateway

Application Zone
  -> internal business services

Data/Platform Zone
  -> Oracle
  -> Kafka
  -> Redis
  -> observability components

External Integration Zone
  -> third-party provider endpoints
```

Actual subnet/firewall design is environment/platform specific and deferred to infrastructure/security implementation.

---

## 7. Production Availability Expectations

Production-critical stateless services should support:

- multiple replicas;
- rolling deployment;
- readiness probes;
- liveness probes;
- graceful shutdown;
- horizontal scaling where justified;
- pod anti-affinity/topology spreading where justified;
- PodDisruptionBudget for critical services.

Exact settings are deferred to deployment and performance epics.
