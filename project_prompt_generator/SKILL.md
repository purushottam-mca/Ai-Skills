# Project-to-Implementation Prompt Generator

## Purpose

Turn a high-level project idea into a **full-fledged, implementation-ready engineering prompt** suitable for a senior/staff-level developer or an AI coding agent.

The input may be only a short project description. The output should transform it into a concrete project specification that covers architecture, implementation, engineering practices, integrations, operations, testing, and extensibility.

The goal is **not** to explain software engineering basics. Assume the implementer already understands programming, Git, APIs, databases, Docker, testing, CI/CD, observability, security fundamentals, and the chosen technology stack.

The generated prompt should be detailed enough that an experienced developer can start building immediately, while still leaving reasonable implementation choices open where the project requirements do not justify over-specification.

---

## Core Instruction

When given a project idea:

1. Understand the actual product/problem being solved.
2. Identify explicit requirements and infer only reasonable engineering requirements.
3. Detect architectural decisions that materially affect the implementation.
4. Ask targeted clarification questions **only when the missing information can change architecture, data model, security, deployment model, API contract, or major technology choices**.
5. If the missing information is non-critical, make a sensible engineering assumption and state it.
6. Produce a complete project-based implementation prompt rather than a generic checklist.
7. Prefer current, maintained technologies and APIs.
8. When the user explicitly asks to look things up, or current documentation materially matters, use authoritative/current documentation—prefer official project documentation and repositories.
9. Avoid blindly following outdated examples, deprecated APIs, or obsolete library versions.
10. Optimize for a project that can be:

* started locally,
* tested locally,
* understood by another engineer,
* extended without architectural rewrites,
* operated and debugged,
* deployed later with minimal restructuring.

---

# Required Output Structure

Generate the implementation prompt using the following structure. Adapt sections when they are irrelevant, but do not omit important engineering concerns merely because the user did not explicitly mention them.

## 1. Project Overview

Define:

* Project name
* Problem being solved
* Target users/consumers
* Primary use cases
* Scope
* Explicit non-goals
* Success criteria

Keep this concrete and implementation-oriented.

---

## 2. Requirements

Separate requirements into:

### Functional Requirements

Describe what the system must do.

Examples:

* CRUD operations
* versioning
* workflows
* authentication
* authorization
* search/filtering
* integrations
* background jobs
* import/export
* notifications
* audit history

### Non-Functional Requirements

Consider:

* performance
* reliability
* scalability
* security
* observability
* maintainability
* extensibility
* portability
* developer experience
* backwards compatibility
* data integrity

Do not invent arbitrary SLAs unless the project requires them. If useful, define reasonable initial targets as assumptions.

---

## 3. Technology Stack

Choose a coherent stack based on the project.

Cover relevant layers:

* backend language/runtime
* framework/router
* database
* cache
* message queue/event system if required
* ORM/query builder/database driver
* frontend if applicable
* API protocol
* authentication
* authorization
* configuration/secrets
* containerization
* CI/CD
* logging
* metrics
* tracing
* documentation

### Version Policy

Do not hard-code old versions merely because they are familiar.

If versions matter:

* verify current stable/recommended versions,
* prefer official documentation,
* use compatible versions across components,
* pin versions in reproducible development environments.

---

## 4. Architecture

Define the proposed architecture at a level appropriate for implementation.

Include:

* major components
* responsibilities
* dependencies
* request flow
* asynchronous flows
* external integrations
* persistence boundaries
* cache boundaries
* security boundaries

Prefer modular architecture over unnecessary microservices.

For a normal application, default to a **modular monolith** unless scale, isolation, deployment independence, or organizational boundaries justify microservices.

Explicitly explain important architectural decisions.

---

## 5. Repository / Project Structure

Provide a practical repository layout.

Example style:

```text
project/
├── cmd/
├── internal/
│   ├── api/
│   ├── domain/
│   ├── service/
│   ├── repository/
│   ├── middleware/
│   └── ...
├── migrations/
├── tests/
├── deploy/
├── docs/
├── scripts/
├── docker-compose.yml
├── Dockerfile
├── Makefile
└── README.md
```

Adapt this to the selected language and architecture.

Explain the responsibility of important directories.

Avoid creating directories solely for appearance.

---

## 6. Domain Model and Data Model

Define the core entities and their relationships.

For each important entity include:

* purpose
* key fields
* identifiers
* timestamps
* status/state
* ownership
* relationships
* indexes
* uniqueness constraints
* soft-delete requirements if applicable
* audit requirements
* versioning requirements

Think about:

* transactional consistency
* concurrent writes
* optimistic locking
* idempotency
* migrations
* historical data
* rollback semantics

If versioning is a core requirement, explicitly distinguish:

* current state,
* immutable historical versions,
* published/active version,
* draft state,
* rollback behavior.

---

## 7. API Design

Define the API contract at implementation level.

Include:

* endpoint/resource structure
* HTTP methods where applicable
* request/response conventions
* pagination
* filtering
* sorting
* validation
* error format
* status codes
* authentication
* authorization
* idempotency
* versioning
* rate limiting where applicable

Prefer REST, gRPC, GraphQL, or another protocol based on actual requirements rather than habit.

For public or AI-facing APIs, define stable contracts and machine-readable schemas.

---

## 8. Authentication and Authorization

Treat security as a first-class design concern.

Consider:

* authentication mechanism
* user/service identities
* roles
* permissions
* resource ownership
* administrative operations
* API keys/service accounts
* token handling
* secret storage
* audit logging

Use least privilege.

If users can modify production behavior/configuration/prompts/tools, explicitly define who can:

* create,
* edit,
* approve,
* publish,
* rollback,
* delete,
* administer.

---

## 9. Versioning and Change Management

For systems involving prompts, configuration, schemas, workflows, policies, templates, or other runtime-controlled artifacts, define a robust versioning model.

Cover:

* immutable versions
* draft vs published versions
* semantic or sequential version identifiers
* publishing
* rollback
* compare/diff
* change history
* author
* timestamp
* reason/change message
* approval workflow if appropriate
* concurrent editing
* validation before publishing

The system should allow runtime changes without requiring an application redeployment when that is part of the product goal.

---

## 10. Runtime / Execution Layer

If the project executes dynamic content, define the runtime behavior.

Examples:

* prompt execution
* plugin/tool execution
* agent workflows
* background processing
* job queues
* external API calls
* retries
* timeouts
* circuit breakers
* concurrency limits
* cancellation
* streaming
* result persistence

Clearly separate:

* configuration/control plane
* runtime/data plane

when the project benefits from that distinction.

---

## 11. AI / LLM Requirements

If AI/LLM functionality is involved, explicitly address:

### Model abstraction

Create a provider-neutral interface where practical.

Support:

* provider
* model
* credentials/configuration
* generation parameters
* streaming
* structured output
* tool/function calling
* token usage
* latency
* errors
* retries

Avoid coupling business logic directly to one provider SDK.

### Prompt Templates

Support:

* variables
* validation
* system/user/developer messages as appropriate
* structured templates
* versioning
* rendering
* escaping/safety considerations
* metadata
* model configuration
* environment-specific configuration

### Agent Skills

If reusable skills are required, define:

* skill identity
* description
* inputs
* outputs
* instructions
* dependencies
* versioning
* permissions
* discoverability
* execution contract

### Tools / Schemas

Use structured schemas where possible.

Define:

* tool name
* description
* input schema
* output schema
* validation
* permissions
* timeout
* error contract
* version compatibility

### MCP

If MCP is requested:

* follow the current MCP specification/documentation,
* expose only intentionally supported capabilities,
* define resources/tools/prompts as appropriate,
* validate arguments,
* enforce authorization,
* provide stable schemas,
* handle errors predictably,
* avoid exposing arbitrary internal APIs.

---


The final result should be specific to the supplied project idea, not a generic restatement of this template.
