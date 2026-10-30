# Contracts for feature extension

## What each boundary promises

Treat a contract as observable semantics as well as a method signature. Record inputs, identity source, outputs, error meanings, side effects, atomicity, idempotency, concurrency, cancellation, ordering and lifetime where relevant. State implementation choices separately. Derive existing guarantees from source/tests; identify unknown guarantees explicitly rather than inventing them.

| Boundary | Agreement | Permitted local evolution |
| --- | --- | --- |
| Client → application | HTTP method/path, request and response shape, validation, authentication and error semantics | Controller internals while preserving external behavior |
| Application → service | Use-case input, trusted operator identity, result and business errors | HTTP presentation without moving business decisions into controllers; service evolution compatible with callers |
| Service → repository/adapter | Business data operation and its semantics, including transaction or atomic consumption requirements | SQL, indexing, caching and storage implementation while preserving the agreed behavior |
| Composition → components | Required dependencies, concrete selection, creation scope, initialization failure and resource ownership | Selecting implementations or adding dependencies at the documented wiring boundary |

Explain how extension follows these agreements, not simply that interfaces exist. An interface hides implementation choices only to the extent its types and semantics permit. IAdaptor is a resource-provider contract; IAdminUser and IVerify are capability contracts. A provider does not automatically register arbitrary repositories or make storage interchangeable.

## Extension workflow

1. Define the new use case and acceptance behavior. Identify existing contracts that already support it before adding methods or layers.
2. Specify the smallest new capability and semantic guarantees. Extend an existing cohesive interface or add a focused one based on consumers and implementations, not a blanket rule.
3. List affected callers, implementers, generated queries, test doubles, routes and constructors. Check compile-time import cycles when placing shared DTO/DO/contract types.
4. Implement the adapter and map persistence data/errors. Put use-case rules and orchestration in the service, HTTP binding/presentation in the controller, and register route and dependencies explicitly.
5. Verify the new behavior and existing contracts at the changed boundary. Use contract tests across alternate adapters when substitutability matters; test failure, concurrency and transactional behavior when required by the feature.
6. Explain concrete change impact and any unavoidable compatibility break. Do not promise that changing storage never changes business code if ORM types/errors leak through the contract.

## Worked extension: administrator list with pagination

Use this as a proposal, not a claim that the reference project already implements it.

| Area | Proposed addition/change | Contract reasoning |
| --- | --- | --- |
| API/DTO | List endpoint and validated page/filter DTO; stable list/total response | Establish public pagination, filtering and sorting semantics |
| Service | ListUsers use case; authorize operator and normalize supported inputs | Keep access policy and use-case rules independent of HTTP and SQL |
| Repository interface/DO | ListUsers(ctx, query) returning records/total/error | Define stable ordering, limits, visibility, empty result and total consistency semantics |
| Concrete repository | GORM Gen filter/count/list queries | Implement agreed semantics; SQL and indexes stay local |
| Wiring | Reuse existing repository instance if adding a method; register new route; inject any newly introduced collaborator | Interfaces alone do not expose the endpoint or construct dependencies |
| Verification | Update all implementers/fakes; test page boundaries, filters, access denial and error mapping | Preserve existing user creation/update behavior and validate the new contract |

State the tradeoff: adding ListUsers to IAdminUser changes the method set and requires every implementer to satisfy it. A separate listing interface may reduce impact when consumption differs, at the cost of additional wiring. Choose according to actual scope.

## Other extension shapes

- New Redis caching strategy: keep service calls stable if the repository contract is unchanged; preserve freshness, invalidation and failure semantics. Choose a decorator or repository-local cache only when justified.
- New storage implementation: implement the same capability interface and choose it during construction. Translate storage-specific errors and models if the boundary requires isolation. Identify constructor changes explicitly.
- New transport such as CLI/RPC: call the existing service using a transport-neutral context and inputs; add transport mapping and identity handling. Check whether current Service actually depends on Gin or HTTP before promising reuse.
- New coupon/payment capability: introduce its own cohesive contract and implementation, inject it, and let the service orchestrate the use case. Define retries, idempotency and cross-system consistency before treating the feature as complete.
- Atomic captcha consumption: a method that consumes a value must promise single-consumer behavior if required. GET followed by DEL does not establish that guarantee; changing implementation should fulfill the declared semantics and be verified under concurrency.

## Reporting standard

For every extension, answer: Which contract permits it? Who must change? Who can remain compatible? Where is the implementation wired? What proves semantic compatibility? Explain limitations after describing the separation's concrete value. Do not infer complete OCP, plugin discovery or zero-change replacement merely from the presence of interfaces.
