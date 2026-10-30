---
name: backend-skill
description: Analyze, design, and implement Go backend architecture using the user's preferred layered approach. Use for requests about Go backend projects, adaptor and repository interfaces, application/business/data layer decoupling, dependency wiring, request flows, feature extension contracts, frontend integration, OpenAPI documentation, microservice evolution, and Gin/GORM/Redis service design.
---

# Backend architecture skill

Use the three-layer separation as a practical starting point: application/transport handles requests and responses; business services coordinate use cases and depend on data capabilities; adaptor/repository implements persistence and external services. Trace actual code and follow existing project conventions; do not impose a directory layout blindly.

## Analyze existing code

1. Read entry points, config, constructors, routes, controllers, services, interfaces, implementations, model/query and infrastructure. Identify precise source paths and revisions when available.
2. Explain the benefits of separation first: which layer owns each responsibility, what can change locally, and which interface mediates each boundary. Distinguish layer responsibilities, compile-time imports, runtime calls, and construction. An interface can decouple invocation even if construction still couples concrete implementations.
3. Map each interface: declaring package, consumer, method contract, concrete implementer, constructor and lifecycle, data and errors crossing it, and substitutions it enables. Examine function callbacks as well as Go interfaces. Describe adaptor resource provider separately from repository and external-service adapters.
4. Trace both object construction and a real request end to end. Show DTO, business data, persistence model and response transformations; identify authentication context and error handling. Ground claims in actual code, marking inferred intent and placeholders separately.
5. Assess actual coupling precisely: imports, concrete return types, ORM errors, constructor selection and interface scope. Explain the practical change or testing impact without dismissing the separation already achieved. Do not equate generated models or dependencies with completed business functionality.
6. For detailed analysis, use compact diagrams or tables and linked source files, not merely a directory inventory. Provide evidence and practical tradeoffs.

## Design and implementation

Respect existing architecture. For new features, place HTTP binding and response handling in application layer, business rules and orchestration in services, and SQL/Redis/third-party details in adapters. Define contracts around required business capabilities, inject implementations at an appropriate composition boundary, and carry context and meaningful errors across boundaries. Use DTO/DO/model conversions where boundaries differ; do not introduce redundant abstractions merely for symmetry. Consider transaction ownership, concurrency, validation, observability and security where relevant. Verify behavior with appropriate tests, including boundary substitution where valuable.

Read `references/layered-design.md` for the reference project's design and concrete evaluation questions.

## Functional extension contracts

Read `references/extension-contracts.md` before implementing or evaluating feature extension. Explain interfaces as agreements that permit layers to evolve: identify the semantic promise, implementation freedom and change impact. For each proposed extension, provide a change matrix covering application, service, contract, adapter and wiring; specify what is added, what changes and why. Include at least one concrete feature walkthrough when asked to explain this architecture. Preserve existing behavior unless the requested feature intentionally changes it. Distinguish interface signature compatibility from behavioral compatibility; adding a method to a Go interface can break every implementer. Verify affected fakes and alternate implementations as well as production code. Never claim an extension point exists without inspecting construction and registration paths.

## Frontend integration and API documentation

Read `references/frontend-api-contracts.md` when analyzing public APIs, developing backend features, or integrating a frontend. Deliver the actual endpoint documentation and machine-readable OpenAPI contract alongside feature implementation; a promise to document later or an endpoint-name list is insufficient. For architecture-only tasks, document inspected representative endpoints and identify unknowns. Trace frontend request/client → gateway or direct HTTP entry → controller → service → adapter → response. Cover authentication, field semantics, error handling, pagination and compatibility. Preserve actual existing behavior and distinguish proposed improvements. Verify contract/code alignment and document how frontend developers generate types, mock APIs and run integration checks.

## Microservice evolution

Read `references/microservice-evolution.md` for future service splitting or distributed implementation. Start with business ownership and module boundaries; map current local interfaces to consumer ports, remote clients, provider APIs and provider services. Explain which dependencies change at construction and which semantics change across a network. Deliver a staged extraction plan, ownership and call diagram, service/API/event contracts, consistency and failure strategy, deployment/observability plan, and verification/rollback criteria at a depth appropriate to the task. Keep external frontend contracts stable through gateway/BFF routing where feasible. Use current official documentation before selecting version-sensitive frameworks, APIs or deployment syntax.
