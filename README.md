

# ContractAdvisor

> **Intelligent API & Event Contract Governance Platform**

ContractAdvisor is a developer platform for detecting, analyzing, and managing API and event contract changes across distributed systems.

Modern systems often contain hundreds or thousands of microservices communicating through REST APIs, Kafka topics, asynchronous events, OpenAPI specifications, and event schemas.

A seemingly small contract change can therefore affect multiple downstream consumers.

ContractAdvisor aims to answer one critical question:

> **What will be affected if this contract changes?**

The platform combines deterministic contract analysis, dependency intelligence, graph-based blast-radius analysis, policy enforcement, contract testing, and AI-assisted migration planning.

---

## Problem

Consider a Payment API with the following request:

```json
{
  "orderId": "123",
  "amount": 500
}
```

A developer changes the contract to:

```json
{
  "orderId": "123",
  "paymentAmount": 500
}
```

Although this appears to be a simple field rename, multiple services may still depend on the `amount` field.

For example:

```text
                    payment-service
                           |
                    PaymentRequest
                           |
                amount -> paymentAmount
                           |
              +------------+------------+
              |            |            |
              v            v            v
        order-service  refund-service  reconciliation-service
```

ContractAdvisor is designed to detect the breaking contract change, discover affected consumers, calculate the potential blast radius, collect supporting evidence, assess risk, and eventually recommend a safe migration strategy.

---

# Core Design Principle

ContractAdvisor follows one fundamental architectural principle:

> **Deterministic first. AI second.**

Contract compatibility, dependency discovery, policy evaluation, graph traversal, and test results should be determined using deterministic systems wherever possible.

AI is introduced only after reliable evidence has been collected.

```text
Contract Change
      |
      v
Deterministic Contract Analysis
      |
      v
Dependency Intelligence
      |
      v
Blast Radius Analysis
      |
      v
Policy & Contract Tests
      |
      v
Evidence
      |
      v
AI Reasoning
      |
      v
Explanation & Migration Plan
      |
      v
Human Approval
```

The LLM is not the source of truth for service dependencies or contract compatibility decisions.

---

# Core Capabilities

## 1. Contract Registry

Maintain versioned API and event contracts belonging to services.

Initial contract formats:

- OpenAPI
- AsyncAPI
- JSON Schema

Example:

```text
Payment API
   |
   +-- v1
   |
   +-- v2
   |
   +-- v3
```

ContractAdvisor tracks which versions exist and which services depend on them.

---

## 2. Breaking Change Detection

ContractAdvisor will deterministically detect incompatible contract changes.

Examples include:

- removed fields
- renamed fields
- field type changes
- optional fields becoming required
- required fields being removed
- removed API endpoints
- removed HTTP methods
- incompatible response changes
- incompatible Kafka event schema changes
- enum changes

Example:

```text
PaymentRequest

Before:
amount: number

After:
paymentAmount: number

Result:
BREAKING CHANGE
```

---

## 3. Dependency Intelligence

ContractAdvisor models relationships between:

- services
- APIs
- API endpoints
- Kafka topics
- event schemas
- repositories
- deployments
- producers
- consumers

Dependencies may be supported by evidence collected from:

- source code
- application configuration
- OpenAPI specifications
- AsyncAPI specifications
- Kafka metadata
- repository analysis
- platform metadata
- runtime telemetry

Example:

```text
order-service
     |
     | CALLS
     v
POST /payments
     |
     | EXPOSED_BY
     v
payment-service
```

---

## 4. Evidence-Based Dependencies

ContractAdvisor should never claim that a dependency exists without supporting evidence.

Example:

```text
Dependency:

order-service
      |
      v
payment-service

Evidence:

Source Code Reference     HIGH
Runtime Trace             HIGH
Configuration             MEDIUM
Naming Convention         LOW
```

This allows ContractAdvisor to distinguish between:

```text
Confirmed Consumer
```

and:

```text
Potential Consumer
```

---

## 5. Blast Radius Analysis

ContractAdvisor uses a dependency graph to determine which services may be affected by a contract change.

Example:

```text
                  payment-service
                        |
                 PaymentResponse
                        |
          +-------------+-------------+
          |             |             |
          v             v             v
     order-service  refund-service  analytics-service
          |
          v
 notification-service
```

Example result:

```text
Direct impact:       3 services
Indirect impact:     1 service

Confirmed consumers: 3
Potential consumers: 1

Risk: HIGH
```

---

## 6. Policy Governance

ContractAdvisor will support configurable engineering governance rules.

Examples:

```text
Breaking changes require a major version increment.

Required fields cannot be removed.

Deprecated fields must remain available during the migration period.

Production APIs must have contract tests.

Kafka event schemas must maintain backward compatibility.
```

Policies are evaluated deterministically.

---

## 7. Contract Testing

ContractAdvisor will support contract verification as part of the change-analysis workflow.

Example pipeline:

```text
Pull Request
     |
     v
ContractAdvisor
     |
     v
Detect Contract Changes
     |
     v
Find Consumers
     |
     v
Run Contract Tests
     |
     v
Risk Analysis
     |
     v
GitHub PR Report
```

Example PR result:

```text
ContractAdvisor Analysis

Backward compatible:
8 changes

Potentially breaking:
2 changes

Affected consumers:
4

Contract tests:
37 passed
2 failed

Risk:
HIGH
```

---

# AI-Assisted Analysis

The AI layer is introduced after deterministic analysis has produced reliable evidence.

The AI system can use information such as:

- contract differences
- affected services
- dependency evidence
- Git history
- source code
- deployment history
- test results
- previous migration plans
- architecture documentation
- incident history

The AI layer can then:

- explain why a change is dangerous
- summarize the impact
- recommend migration strategies
- inspect repository context
- suggest code changes
- generate migration plans

Example:

```text
Recommended Migration

1. Introduce paymentAmount.
2. Keep amount temporarily for backward compatibility.
3. Deploy updated producer.
4. Migrate downstream consumers.
5. Verify runtime usage.
6. Deprecate amount.
7. Remove amount in the next major version.
```

---

# Agentic AI

Future versions of ContractAdvisor will provide an AI agent with controlled tools.

Example tools:

```text
getApiContract()

getEventSchema()

findConsumers()

searchRepository()

analyzeGitDiff()

getDeploymentHistory()

runContractTests()

generateMigrationPatch()

createMigrationPR()
```

Agent execution will follow:

```text
READ
  |
  v
ANALYZE
  |
  v
PROPOSE
  |
  v
HUMAN APPROVAL
  |
  v
EXECUTE
```

Mutating actions such as creating migration pull requests will require explicit approval.

---

# High-Level Architecture

```text
                         Developer
                             |
                             v
                     GitHub / GitLab
                             |
                         Webhook
                             |
                             v
                    ContractAdvisor Platform
                             |
                          Kafka
                             |
       +---------------------+---------------------+
       |                     |                     |
       v                     v                     v
Contract Engine      Dependency Engine       Runtime Engine
       |                     |                     |
       +---------------------+---------------------+
                             |
                             v
                       Impact Engine
                             |
                 +-----------+-----------+
                 |                       |
                 v                       v
           Policy Engine           Contract Tests
                 |                       |
                 +-----------+-----------+
                             |
                             v
                       Evidence Pack
                             |
                             v
                      AI Orchestrator
                             |
                 +-----------+-----------+
                 |           |           |
                 v           v           v
                LLM         RAG         Tools
                 |           |           |
                 +-----------+-----------+
                             |
                             v
                      Migration Plan
                             |
                             v
                      Human Approval
                             |
                             v
                         GitHub PR
```

---

# Technology Stack

## Backend

- Java
- Spring Boot
- Spring Data JPA
- PostgreSQL
- Apache Kafka
- Neo4j

## Frontend

- Angular
- TypeScript

## Contract Standards

- OpenAPI
- AsyncAPI
- JSON Schema

## AI — Planned

- Amazon Bedrock
- Spring AI
- Retrieval-Augmented Generation (RAG)
- Tool-enabled AI workflows
- Structured LLM outputs

## Infrastructure

- Docker
- Docker Compose
- Kubernetes
- AWS
- Terraform
- GitHub Actions

## Observability — Planned

- OpenTelemetry
- Prometheus
- Grafana
- AWS CloudWatch

---

# Repository Structure

```text
contract-advisor/
|
+-- backend/
|   |
|   +-- ContractAdvisor Java / Spring Boot backend
|
+-- frontend/
|   |
|   +-- Angular dashboard
|
+-- infrastructure/
|   |
|   +-- Docker
|   +-- Terraform
|   +-- Kubernetes
|   +-- deployment configuration
|
+-- test-services/
|   |
|   +-- Reference microservices used to test
|       contract and dependency analysis
|
+-- docs/
|   |
|   +-- Architecture documentation
|   +-- diagrams
|   +-- Architecture Decision Records
|
+-- README.md
```

---

# Reference Microservices

ContractAdvisor will be developed against a controlled reference microservices ecosystem.

Planned services include:

```text
order-service
payment-service
refund-service
inventory-service
notification-service
analytics-service
fraud-service
loyalty-service
```

These services will expose intentionally connected REST APIs and Kafka events.

Example:

```text
                   order-service
                    /         \
                   /           \
                  v             v
          payment-service   inventory-service
             /      \
            v        v
   refund-service   fraud-service
                         |
                         v
                notification-service
```

Kafka topics may include:

```text
order.events
payment.events
inventory.events
```

Breaking changes will intentionally be introduced into these services to validate ContractAdvisor.

Example scenarios:

- remove a required field
- rename a field
- change a field type
- make an optional field required
- remove an endpoint
- remove a Kafka event field
- change an event schema
- keep a consumer on an older contract version
- introduce stale dependency information

---

# Planned Backend Modules

The initial backend will follow a modular architecture.

```text
backend/
|
+-- catalog
|
+-- contracts
|
+-- analysis
|
+-- dependency
|
+-- impact
|
+-- policy
|
+-- integration
|
+-- security
|
+-- common
```

The project will not begin with unnecessary microservices.

Modules may later become independently deployable services when scaling or operational requirements justify the separation.

---

# Data Architecture

## PostgreSQL

PostgreSQL will act as the primary system of record.

It will store information such as:

```text
Services

Repositories

Contracts

Contract Versions

Dependencies

Evidence

Change Sets

Changes

Impact Analyses

Risk Assessments

Policies

Policy Violations

Test Runs

AI Analyses

Migration Plans
```

---

## Neo4j

Neo4j will maintain the dependency graph used for impact analysis.

Example graph:

```text
(Service)
    |
   OWNS
    |
    v
  (API)
    |
 EXPOSES
    |
    v
(Endpoint)
    |
CONSUMED_BY
    |
    v
(Service)
```

Kafka example:

```text
(Service)
    |
 PUBLISHES
    |
    v
(KafkaTopic)
    |
USES_SCHEMA
    |
    v
(EventSchema)
    |
CONSUMED_BY
    |
    v
(Service)
```

Graph traversal will be used to calculate the potential blast radius of contract changes.

---

# Event-Driven Architecture

ContractAdvisor will use Kafka to process asynchronous analysis workflows.

Planned events include:

```text
ContractChangedEvent

ImpactAnalysisRequestedEvent

ImpactAnalysisCompletedEvent

PolicyEvaluationRequestedEvent

PolicyViolationDetectedEvent

ContractTestRequestedEvent

ContractTestCompletedEvent

AIAnalysisRequestedEvent

AIAnalysisCompletedEvent

MigrationPlanGeneratedEvent
```

The event architecture will demonstrate concepts such as:

- partitioning
- consumer groups
- retries
- dead-letter queues
- idempotency
- correlation IDs
- event schema evolution
- distributed tracing

---

# Reliability

ContractAdvisor is designed to eventually support:

```text
Idempotent processing

Retry with backoff

Dead-letter queues

Correlation IDs

Distributed tracing

Timeout handling

Failure recovery

Duplicate webhook handling

Event replay

Graceful degradation
```

For example, duplicate GitHub webhook events should not result in duplicate analyses.

---

# Security

Planned security capabilities include:

- OAuth2 / OpenID Connect
- Role-Based Access Control
- GitHub App authentication
- short-lived repository access tokens
- AWS Secrets Manager
- AWS KMS
- IAM roles
- audit logging
- least-privilege access

AI-generated changes will require human approval before repository mutations are performed.

---

# Development Roadmap

## Phase 1 — Foundation

- Repository setup
- Spring Boot backend
- Angular frontend
- PostgreSQL
- Docker Compose
- CI pipeline

## Phase 2 — Contract Intelligence

- Service catalog
- Contract registry
- OpenAPI parsing
- contract versioning
- deterministic contract diff engine

## Phase 3 — Dependency Intelligence

- dependency evidence model
- Neo4j dependency graph
- consumer discovery
- graph traversal
- blast-radius analysis

## Phase 4 — Event-Driven Analysis

- Kafka workflows
- event schemas
- partitioning
- idempotency
- retries
- dead-letter handling
- correlation IDs

## Phase 5 — Governance

- policy engine
- deterministic risk assessment
- contract testing
- GitHub pull-request reporting

## Phase 6 — Runtime Intelligence

- OpenTelemetry
- runtime dependency evidence
- deployment metadata
- Kafka consumer activity

## Phase 7 — AI Intelligence

- evidence-grounded explanations
- RAG
- migration planning
- controlled tool calling
- structured AI output

## Phase 8 — Agentic Remediation

- code-change proposals
- automated validation
- migration PR generation
- human approval workflows

## Phase 9 — Production Hardening

- Kubernetes
- AWS deployment
- Terraform
- observability
- security hardening
- performance testing
- failure testing
- cost analysis

---

# Current Development Status

> **Under active development**

Current milestone:

### Walking Skeleton

```text
OpenAPI Contract Change
          |
          v
Deterministic Contract Diff
          |
          v
Dependency Discovery
          |
          v
Blast-Radius Analysis
```

The first target scenario is detecting a breaking change to the Payment API and identifying its downstream consumers.

---

# First Target Scenario

Original Payment API contract:

```json
{
  "orderId": "123",
  "amount": 500
}
```

Modified contract:

```json
{
  "orderId": "123",
  "paymentAmount": 500
}
```

Expected ContractAdvisor analysis:

```text
BREAKING CHANGE

Contract:
Payment API

Change:
PaymentRequest.amount REMOVED

Affected Consumers:

order-service
refund-service
reconciliation-service

Risk:
HIGH

Evidence:

order-service           HIGH confidence
refund-service          HIGH confidence
reconciliation-service  MEDIUM confidence
```

---

# Engineering Goals

ContractAdvisor is also being built as a practical exploration of:

- Java backend engineering
- Spring Boot
- distributed systems
- event-driven architecture
- Apache Kafka
- graph databases
- graph algorithms
- REST API design
- API contract evolution
- PostgreSQL
- AWS architecture
- Docker
- Kubernetes
- Infrastructure as Code
- observability
- reliability engineering
- system design
- AI-assisted software engineering
- Large Language Models
- Retrieval-Augmented Generation
- AI agents

The project intentionally prioritizes architectural reasoning and measurable engineering trade-offs over unnecessary technology complexity.

---

# Project Philosophy

ContractAdvisor is not intended to become:

> "A collection of technologies placed into one project."

Every technology introduced into the platform should solve a specific engineering problem.

Examples:

```text
PostgreSQL
    -> transactional system of record

Neo4j
    -> dependency relationships and graph traversal

Kafka
    -> asynchronous distributed analysis workflows

OpenAPI / AsyncAPI
    -> machine-readable contracts

OpenTelemetry
    -> runtime dependency evidence and tracing

LLMs
    -> evidence-grounded explanation and migration reasoning

Kubernetes
    -> deployment, scaling and operational resilience

Terraform
    -> reproducible cloud infrastructure
```

The goal is not maximum architectural complexity.

The goal is to build a system whose architecture can be explained and defended.

---

# License

License information will be added as the project matures.

