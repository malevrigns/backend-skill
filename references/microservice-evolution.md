# Evolving layered Go backends into services

## Boundary and topology decisions

Identify business capabilities, owners, data ownership, transactional invariants, deployment/scaling needs and actual operational capacity. Prefer explicit modules in the current monolith until independent deployment has a concrete benefit. Candidate domains such as identity, catalog, order and payment are proposals requiring validation. Never split controllers, services and repositories into separate network services merely because they are different layers. Each extracted service retains its own application, business and data layers.

Draw service ownership, public entry and synchronous/asynchronous dependencies. Provide an ownership table: capability, owner, owned data, consumers, API/events, consistency expectations and reason for extraction. Treat IAdaptor as local resource wiring, not service discovery or a remote protocol.

## How a local interface becomes a remote boundary

Example proposal: order placement needs catalog availability and price.

Before: OrderService → Catalog capability interface → local Catalog business implementation → Catalog repository → DB.
After: OrderService → consumer CatalogPort → Catalog HTTP/gRPC client adapter → provider transport handler → CatalogService → provider repository → provider-owned data.

Define a business capability port such as GetPurchasableItem(ctx, itemID), with a snapshot result and explicit unavailable/not-found/invalid-state errors; inspect real requirements before choosing its signature. Do not mechanically remote a table repository or expose GORM models. A price read alone does not reserve inventory or guarantee checkout price; specify price validity, reservation/confirmation and concurrency semantics if relevant. Map network DTO/protobuf to consumer business types. Change dependency wiring from a local implementation to a remote client and document timeout/error behavior. Identify any business logic that must change due to distributed semantics.

For HTTP use OpenAPI; for gRPC provide versioned protobuf messages/services and compatibility rules; for asynchronous communication document event schemas and delivery semantics. Select protocols based on consumers, streaming and tooling. A Go interface is an in-process contract and cannot itself be a wire protocol. Do not claim replacing its implementation automatically solves distributed transactions, discovery or failures.

## Public frontend contract

Keep browser-facing APIs documented with OpenAPI. Introduce gateway/BFF routing when useful so the frontend can retain base URL and response semantics during extraction. Define authentication at the edge and authorization in the owning service, propagate verified identity and trace context securely, and avoid trusting arbitrary caller-provided user headers. Distinguish public compatibility from internal contract versioning. Document client-visible partial failure and asynchronous progress when workflows change.

## Data and consistency

Assign one authoritative writer per business entity. Enforce module ownership before physical database separation; do not assume separate databases alone establish boundaries. Remove cross-service direct table access and joins; use owner APIs, snapshots, aggregation or event-built read models. Specify data migration, backfill, change capture or dual-write consistency, cutover and rollback; avoid uncoordinated dual writes.

Keep local invariants in local transactions. For order/payment/entitlement workflows specify state machines, durable intent (for example transactional outbox when justified), event publication, inbox/deduplication, retry/backoff, dead-letter handling, reconciliation and business compensation. Define event ID, type/version, occurrence time, aggregate ID/version, correlation/trace context, payload and producer/consumer ownership. Handle duplicate and out-of-order delivery; do not promise exactly-once execution from a broker alone. Use immutable order price snapshots and documented business rules where appropriate.

## Network and operational contracts

Set end-to-end deadlines and bounded per-hop timeouts; propagate cancellation. Retry only safe/idempotent operations with limited attempts, backoff and jitter; avoid retries at every layer. Define idempotency key scope, request matching, persistence and result replay for sensitive commands. Add load protection, connection management and circuit breaking when justified. Specify explicit fallbacks; never hide payment or inventory failures as success.

Choose service addressing/discovery and load balancing to match the platform; do not require Kubernetes, etcd or a mesh by default. Provide configuration/secrets, independent build and deployment, readiness/liveness, graceful shutdown, resource limits and database migration ownership. Plan structured logs, metrics, distributed tracing, trace propagation and actionable alerts. For service-to-service access use appropriate network identity and authorization. Preserve contract compatibility across rolling deployments.

## Staged extraction and proof

1. Baseline behavior and frontend contract. Locate cross-module imports, shared tables, transactions and resource constructors.
2. Enforce module boundaries and consumer contracts in-process; add meaningful behavior/contract checks.
3. Implement provider HTTP/RPC handler and consumer client, plus service-owned data access and migration plan.
4. Deploy the provider, verify contracts and failure behavior, migrate data, then route a controlled portion of traffic where feasible. Avoid shadowing write side effects.
5. Cut over authoritative ownership and routing, verify correctness and operational targets, retain a bounded rollback path. Resolve data written after cutover before rollback; routing rollback alone may be insufficient.
6. Remove obsolete local code/shared writes only after proving the new path.

Deliver: current/target topology; module/service and data ownership; local-interface-to-remote-contract mapping; API/protobuf/event schemas for the requested scope; changed files/wiring; migration stages; timeout/retry/idempotency/consistency policies; verification evidence and rollback criteria. Test serialization, error mapping, authorization, duplicates, timeouts, partial failures, compatibility and user-visible behavior relevant to the feature. Clearly label proposals versus implemented/running infrastructure.
