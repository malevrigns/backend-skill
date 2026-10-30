# Layered Go backend: reference and analysis

Reference: https://github.com/yuanqinguo/flyfei/tree/master/src/golang/project/backend/mall (inspect current source before asserting current behavior).

## Separation in the reference

- Application: Gin router and middleware handle routing, identity and cross-cutting concerns; API controllers bind HTTP, obtain identity, call services, and format responses.
- Business: admin Service coordinates user operations and captcha use cases. It holds `IAdminUser` and `IVerify` capability interfaces, enabling business calls without embedding SQL or Redis commands.
- Data/infrastructure: adaptor/repo implements user operations with GORM Gen and MySQL; adaptor/redis implements captcha storage with Redis. Model and generated query correspond to database structure and query API.
- `IAdaptor` aggregates config, GORM DB and Redis clients for construction; this resource-provider interface differs from the business-facing `IAdminUser` and `IVerify` operations. It returns concrete infrastructure clients, so assess its actual boundary accordingly.
- `IRouter` separates server setup from route registration; injected token and health-check functions separate common control flow from supplied behavior.

Creation chain: main initializes config, DB, Redis and adaptor → router → admin controller → service → user repository, Redis verifier, captcha generator. Request chain: router/middleware → controller → service → repository interface → implementation → GORM Gen → MySQL. Captcha service coordinates captcha generation and Redis verifier.

Creation-user flow: request DTO + authenticated operator → service DO → repository model with audit fields → generated query → ID returned through service and API. Check actual route, identity and transformations before repeating this example.

The design separates concerns and decouples business calls from storage implementation through interfaces. Measure remaining concrete coupling independently: service constructor selects concrete adapters; repository interface exposes generated model; service checks GORM error; controller holds concrete service; IAdaptor exposes concrete clients. These constrain replacement and tests, but do not erase the functional layer separation. Suggest improvements only when justified by the project and user goals.

## Questions for a real project

- What changes independently at each boundary? Which calls pass through an interface, and who chooses and creates its implementation?
- What does each interface promise: state changes, errors, ownership, atomicity, timeouts, context and side effects? Do names accurately reflect these semantics?
- Do DTO, service data and persistence model carry distinct responsibilities? Who creates audit fields and validates the business invariant?
- Are transactions and consistency boundaries explicit across databases, caches and remote calls? Can retries or concurrent requests break the contract?
- Are authentication, authorization, error mapping and observability implemented, or only scaffolded? Check route registration and real implementations.
- What tests can replace an adapter, and does the constructor actually allow injection? Explain any extra setup required.
