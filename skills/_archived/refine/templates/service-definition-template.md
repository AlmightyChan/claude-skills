# Service Definition: {Service Name}

**Status:** {Draft | Proposed | Approved | Active | Deprecated}
**Owner:** {Team or individual name}
**Date:** {YYYY-MM-DD}
**Version:** {Version number}

---

## 1. Service Overview

### 1.1 Purpose

{Describe the primary business purpose and value this service provides in 2-3 sentences. What problem does it solve?}

### 1.2 Scope

{Define what this service does and what it does NOT do. Clarify boundaries and responsibilities.}

### 1.3 Service Type

{Specify whether this is an API service, microservice, backend service, integration service, etc., and the primary communication pattern (API-driven, event-driven, data streaming).}

> Example: API service, RESTful, synchronous request-response pattern

---

## 2. Out of Scope

{List capabilities, features, or responsibilities explicitly NOT handled by this service. Include common misconceptions or requests that should be directed elsewhere.}

> Example: This service does NOT handle user authentication (delegated to Auth Service), does NOT store user preferences (handled by User Profile Service), does NOT send notifications (handled by Notification Service).

---

## 3. Business Context

### 3.1 Business Driver

{Explain the business need, user story, or strategic initiative that justifies this service's existence. 2-3 sentences describing the "why now" and the business opportunity or pain point addressed.}

### 3.2 Success Metrics

{Define measurable business outcomes (e.g., reduced processing time, improved user satisfaction, increased conversion rate) that indicate the service is delivering value. Include 3-5 metrics with target values.}

### 3.3 Stakeholders

{List key stakeholders, their roles, and their interest in this service. Format: Name/Team, Role, Primary Interest.}

---

## 4. API Contracts

### 4.1 Protocol & Format

{Specify communication protocol (REST, gRPC, GraphQL, message queue) and data format (JSON, Protocol Buffers, XML).}

### 4.2 Base URI

{Document the base endpoint structure (e.g., `/api/{service-name}/v{version}/`).}

### 4.3 Resources & Endpoints

{List all resources with their endpoints, HTTP methods, input parameters, expected responses, and error codes. Use OpenAPI/Swagger specification where applicable. Format each as: Method + Path, Request Body Schema, Success Response (200/201), Error Responses (400/401/404/500).}

> Example: POST /api/orders/v1/, Request: {userId, items[], shippingAddress}, Success: 201 {orderId, status, estimatedDelivery}, Errors: 400 (invalid items), 401 (unauthorized), 404 (user not found), 500 (processing error).

### 4.4 Versioning Strategy

{Define versioning approach (URI-based, header-based, content negotiation) and backward compatibility policy.}

### 4.5 Authentication & Authorization

{Specify authentication mechanism (OAuth 2.0, API keys, JWT) and authorization model (RBAC, ABAC, ACL).}

---

## 5. Data Model

### 5.1 Domain Entities

{Define core data entities this service owns, their relationships, and lifecycle management. List each entity with key fields, cardinality (one-to-one, one-to-many, many-to-many), and state transitions if applicable.}

> Example: Order (id, userId, status, createdAt) → has many OrderItems → each references Product. Status lifecycle: PENDING → CONFIRMED → SHIPPED → DELIVERED.

### 5.2 Data Schema

{Provide schema definitions (JSON Schema, Protocol Buffers, database schema) for request/response payloads and internal data structures. Include field names, types, constraints (required/optional, min/max, regex), and validation rules.}

### 5.3 Data Ownership

{Clarify which data this service is the source of truth for and which data is consumed from upstream services. Format: Entity Name - Owner (this service / ServiceX), Read/Write access.}

### 5.4 Data Retention & Archival

{Specify retention policies, archival strategies, and compliance requirements (GDPR, HIPAA, SOC2). Include active retention period, archive retention period, deletion policy.}

---

## 6. Dependencies

### 6.1 Upstream Services

{List services this service depends on, including endpoints consumed, data contracts, and failure handling strategies. Format: Service Name, Endpoint(s) Used, Purpose, Failure Strategy (retry/timeout/fallback).}

> Example: UserService, GET /api/users/{id}, fetch user profile data, 3 retries with 100ms timeout, fallback to cached profile.

### 6.2 Downstream Consumers

{Identify services or clients that depend on this service, and document the coupling level (tight/loose). Format: Consumer Name, Endpoints Consumed, Usage Pattern, Coupling (sync/async, tight/loose).}

### 6.3 External Dependencies

{Document third-party APIs, SaaS platforms, databases, message queues, caches, or infrastructure components required. Include vendor, purpose, SLA, and business continuity plan if vendor is unavailable. For critical vendors, document contract renewal dates, pricing change risk, lock-in level (low/medium/high), and alternative vendors evaluated as fallback options.}

### 6.4 Fallback & Circuit Breaker

{Define behavior when dependencies are unavailable (graceful degradation, cached responses, circuit breaker thresholds). Specify circuit breaker open threshold (error rate %), half-open retry logic, and fallback behavior per dependency.}

---

## 7. Service Level Objectives (SLOs) & Agreements (SLAs)

### 7.1 SLIs (Service Level Indicators)

{Define measurable metrics such as uptime, latency (p50, p95, p99), error rate, and throughput.}

### 7.2 SLOs (Service Level Objectives)

{Set internal targets for each SLI (e.g., 99.95% uptime over 30 days, p95 latency < 200ms).}

### 7.3 SLAs (Service Level Agreements)

{Document customer-facing commitments, consequences of breaches, and escalation procedures.}

### 7.4 Error Budget

{Define acceptable failure threshold and policy for when the error budget is exhausted.}

> Example: 0.05% error budget over 30 days; if exhausted, halt feature releases and focus on reliability improvements.

---

## 8. Scalability & Performance

### 8.1 Expected Load

{Provide baseline and peak traffic projections (requests per second, concurrent users, data volume). Include daily average, peak hour, and seasonal/event spikes.}

> Example: Baseline 500 req/sec, peak 5000 req/sec (weekday 9-11 AM), seasonal spike 15000 req/sec (Black Friday).

### 8.2 Scaling Strategy

{Specify horizontal vs. vertical scaling approach, auto-scaling triggers, and limits. Include metric thresholds, scale-out/scale-in policy, min/max instance count.}

### 8.3 Performance Benchmarks

{Document target response times, throughput, and resource utilization under normal and peak conditions. Use percentile notation (p50, p95, p99) for latency.}

### 8.4 Capacity Planning

{Outline how capacity is monitored, forecast, and provisioned to meet future demand. Specify monitoring tools, forecasting model (trend/seasonal), lead time for provisioning additional capacity. Include a unit cost model (cost per transaction or request), monthly operating budget target, and cost overrun alert thresholds that trigger review or scaling adjustments.}

---

## 9. Security & Compliance

### 9.1 Threat Model

{Identify key security risks (data breaches, injection attacks, DDoS, unauthorized access) and mitigation strategies. Format each threat as: Threat Name, Likelihood (Low/Medium/High), Impact (Low/Medium/High), Mitigation Control.}

### 9.2 Data Protection

{Specify encryption at rest and in transit, tokenization, masking, and key management practices. Include algorithms (e.g., AES-256, TLS 1.3), key rotation policy, and PII/sensitive field handling.}

### 9.3 Compliance Requirements

{Document regulatory and organizational standards (PCI-DSS, GDPR, HIPAA, SOC2). List each with specific controls required and validation/audit frequency.}

### 9.4 Audit & Logging

{Define what security events are logged, retention period, and who has access to audit logs. Include events: authentication attempts (success/fail), authorization denials, data access/modification, configuration changes. Specify log format, storage location, access controls.}

---

## 10. Monitoring & Observability

### 10.1 Metrics

{List key operational metrics collected (latency, error rates, request volume, resource utilization) and monitoring tools used. Format: Metric Name, Type (gauge/counter/histogram), Collection Interval, Alert Threshold.}

> Example: http_request_duration_seconds (histogram), 10s interval, alert if p95 > 500ms; error_rate (gauge), 10s interval, alert if > 1%.

### 10.2 Logging Strategy

{Specify log levels, structured logging format, centralized logging solution, and retention policy. Include log level usage (DEBUG/INFO/WARN/ERROR/FATAL), JSON structure with required fields (timestamp, level, service, trace_id, message), aggregation tool, retention period per level.}

### 10.3 Distributed Tracing

{Define tracing implementation (OpenTelemetry, Jaeger, Zipkin) and trace sampling strategy. Specify sampling rate (e.g., 1% baseline, 100% for errors), trace context propagation headers, span naming conventions.}

### 10.4 Alerting & On-Call

{Document alert conditions, escalation paths, runbooks, and on-call rotation responsibilities. Format each alert: Name, Condition, Severity (P0-P3), Notification Channel (PagerDuty/Slack), Escalation (immediate/15min/30min), Runbook Link.}

### 10.5 Health Checks

{Define liveness and readiness probe endpoints, expected response codes, and timeout thresholds. Specify: Liveness endpoint (e.g., /health/live), Readiness endpoint (e.g., /health/ready), expected HTTP 200, timeout (e.g., 5s), check interval (e.g., 10s), failure threshold (e.g., 3 consecutive failures).}

---

## 11. Deployment Strategy

### 11.1 Environment Topology

{Describe deployment environments (dev, staging, production) and their configuration differences. Format: Environment Name, Purpose, Instance Count, Data Isolation (shared/isolated), Key Config Differences (DB endpoint, feature flags, external API endpoints).}

### 11.2 Deployment Pipeline

{Outline CI/CD stages (build, test, deploy), approval gates, and rollback procedures. Specify stage sequence, automated test gates (unit/integration/e2e), manual approval steps, rollback trigger criteria, rollback procedure.}

### 11.3 Deployment Pattern

{Specify blue-green, canary, rolling, or feature-flag-based deployment approach. Include rollout percentage schedule for canary (e.g., 5% → 25% → 50% → 100%), health check validation at each stage, automatic rollback conditions.}

### 11.4 Infrastructure as Code

{Document IaC tools used (Terraform, CloudFormation, Pulumi) and where configuration is stored. Include repository location, state management (local/remote backend), change approval process, secret management approach.}

### 11.5 Containerization

{Specify container runtime (Docker, containerd), orchestration (Kubernetes, ECS, Fargate), and base image strategy. Include base image (e.g., node:20-alpine), image registry, image scanning policy, resource limits (CPU/memory), restart policy.}

---

## 12. Ownership & RACI

### 12.1 Team Ownership

{Identify the team responsible for this service (small, autonomous, full-lifecycle ownership). Include team name, size, key members, and ownership scope (design, development, testing, deployment, operations).}

### 12.2 RACI Matrix

{Define who is Responsible, Accountable, Consulted, and Informed for key activities: development, deployment, incident response, and strategic decisions. Format as table: Activity | Responsible | Accountable | Consulted | Informed.}

> Example: Deployment → R: DevOps Engineer, A: Engineering Manager, C: Tech Lead, I: Product Manager.

### 12.3 Contact Information

{Provide team email, Slack channel, issue tracker, and on-call rotation details. Include primary contact email, Slack channel (#team-name), issue tracker link (Jira/GitHub), on-call rotation tool (PagerDuty), escalation contact.}

---

## 13. Change Management

### 13.1 Versioning Policy

{Define major, minor, and patch version criteria and backward compatibility guarantees. Specify semver interpretation: MAJOR (breaking changes), MINOR (backward-compatible features), PATCH (backward-compatible fixes). State backward compatibility window (e.g., support N-1 major versions).}

### 13.2 Deprecation Process

{Specify how features or versions are deprecated (notice period, migration path, sunset timeline). Include minimum deprecation notice period, deprecation warnings in responses (headers/body), migration guide requirements, sunset timeline.}

### 13.3 Changelog

{Maintain a version history documenting breaking changes, new features, bug fixes, and security patches. Format: Version Number, Release Date, Categories (Breaking/Added/Changed/Deprecated/Removed/Fixed/Security).}

### 13.4 Communication Plan

{Outline how breaking changes are announced to downstream consumers and stakeholders. Specify notification channels (email, Slack, API changelog, release notes), advance notice period, migration documentation requirements, support window for old versions.}

---

## 14. Testing Strategy

### 14.1 Unit Testing

{Define code coverage targets, testing frameworks, and what components are unit tested. Specify minimum coverage threshold (e.g., 80% line coverage), framework (Jest/pytest/JUnit), what is tested (business logic, utilities, data transformations), what is mocked (external dependencies).}

### 14.2 Integration Testing

{Specify how service interactions with dependencies are validated (contract testing, integration test environment). Include testing approach (consumer-driven contracts, integration test suite), test environment (staging/dedicated), mock vs real dependencies, contract validation tool (Pact/Spring Cloud Contract).}

### 14.3 End-to-End Testing

{Describe full workflow validation across services and user journeys tested. List critical user journeys covered (e.g., user registration → login → checkout), test environment, data setup/teardown strategy, frequency (pre-deploy/nightly/weekly).}

### 14.4 Performance & Load Testing

{Document load testing tools, scenarios, and acceptance criteria. Specify tool (JMeter/k6/Gatling), load scenarios (baseline/peak/stress), acceptance criteria (p95 latency, error rate, throughput), test frequency, environment used.}

### 14.5 Chaos Engineering

{Define resilience testing practices (fault injection, dependency failure simulation). Include scenarios tested (service unavailable, high latency, partial failure), tooling (Chaos Monkey, Gremlin), test frequency, rollback criteria.}

---

## 15. Disaster Recovery & Business Continuity

### 15.1 Backup Strategy

{Specify data backup frequency, storage location, encryption, and restoration testing schedule. Include backup type (full/incremental/differential), frequency (continuous/hourly/daily), retention period, storage location (same region/cross-region/offsite), encryption method, restoration testing frequency (monthly/quarterly).}

### 15.2 Recovery Time Objective (RTO)

{Define maximum acceptable downtime before service must be restored. Specify per environment and criticality level.}

> Example: RTO = 4 hours for production; service must be operational within 4 hours of failure.

### 15.3 Recovery Point Objective (RPO)

{Define maximum acceptable data loss measured in time. Specify per data type or transaction class.}

> Example: RPO = 15 minutes; maximum acceptable data loss is 15 minutes of transactions.

### 15.4 Failover Plan

{Document failover triggers, multi-region/availability zone strategy, and manual vs. automated failover procedures. Include failover detection criteria (health check failures, latency thresholds), multi-region setup (active-active/active-passive), DNS/load balancer failover mechanism, data replication strategy, manual approval requirement (yes/no).}

### 15.5 Incident Response

{Outline incident severity levels, response team structure, communication protocol, and post-mortem process. Define severity levels (P0-P4) with SLA response times, incident commander role, communication channels (Slack incident room, status page updates), stakeholder notification triggers, post-mortem requirement criteria, blameless post-mortem template.}

---

## 16. Documentation & Knowledge Management

### 16.1 Architecture Diagrams

{Provide system context, container, and component diagrams (C4 model recommended). Include diagram types, location/links, update frequency, diagramming tool used.}

> Example: C4 Context Diagram (docs/architecture/context.png), Container Diagram (docs/architecture/container.png), updated quarterly or on major architecture changes, created with draw.io.

### 16.2 API Documentation

{Maintain up-to-date API reference (OpenAPI/Swagger, Postman collections, interactive docs). Specify documentation format (OpenAPI 3.0 spec location), interactive docs URL, Postman collection location, documentation generation approach (manual/auto-generated from code), update trigger (every release).}

### 16.3 Runbooks

{Document operational procedures for common tasks (deployment, rollback, scaling, debugging). List runbooks maintained: deployment procedure, rollback procedure, scaling (up/down), common debugging scenarios, incident response checklist. Include location (wiki/docs folder), format (markdown/confluence), review frequency.}

### 16.4 ADRs (Architecture Decision Records)

{Record significant architectural decisions, alternatives considered, and rationale. Specify ADR location (docs/adr/), format template, numbering scheme, review/approval process, retention policy.}

### 16.5 Onboarding Guide

{Create developer onboarding materials covering local setup, testing, deployment, and contribution guidelines. Include sections: prerequisite tools, local environment setup, running tests, deploying to dev/staging, code review process, contribution workflow, team practices. Target: new developer productive within 1 day.}

---

## 17. Risk Assessment & Assumptions

### 17.1 Known Risks

{Document technical, operational, or organizational risks with likelihood, impact, and mitigation strategies. Format each as: Risk Description, Likelihood (Low/Medium/High), Impact (Low/Medium/High), Mitigation Strategy, Contingency Plan, Owner.}

### 17.2 Assumptions

{State explicit assumptions this service definition relies on. Mark each assumption as Validated or Requires Validation.}

> Example: [Requires Validation] Assume third-party payment gateway maintains 99.9% uptime SLA. [Validated] User authentication handled by existing Auth Service with no changes required.

---

## 18. Open Questions

{List unresolved items, pending decisions, or areas requiring further investigation.}

| # | Question | Owner | Target Date | Status |
|---|----------|-------|-------------|--------|
| 1 | {Unresolved question or pending decision} | {Who can answer} | {Date} | {Open/Resolved} |

---

## 19. Verification Checklist

Before submitting this service definition, verify:

- [ ] All required sections are filled with specific, actionable content (not just guidance text)
- [ ] API contracts include concrete endpoints, methods, request/response schemas, and error codes
- [ ] SLOs/SLAs specify measurable thresholds with numeric targets (uptime %, latency ms, error rate %)
- [ ] Dependencies are documented with both upstream services and downstream consumers
- [ ] Security requirements address authentication, authorization, data protection, and compliance
- [ ] Monitoring strategy defines specific metrics, alerts, and health check endpoints
- [ ] Deployment strategy specifies environments, pipeline stages, and rollback procedures
- [ ] Unit cost model documented (cost per transaction/request, monthly budget target, overrun thresholds)
- [ ] Critical vendor business continuity plans exist (renewal dates, pricing risk, alternatives)

---

## 20. Glossary

{Define domain-specific terms, acronyms, or technical concepts used in this document. List alphabetically.}

---

## 21. References

{List related documents, research, external resources, and supporting materials. Use markdown links with descriptive text.}

- Related Services: {Links to related service definitions}
- Design Documents: {Links to ADRs, RFCs, design docs}
- External Standards: {Links to API standards, compliance frameworks, architectural patterns}

---

## 22. Approval & Sign-off

| Role | Name | Date | Status |
|------|------|------|--------|
| {Role} | {Name} | {Date} | {Pending/Approved/Rejected} |
