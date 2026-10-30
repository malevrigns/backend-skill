# Frontend integration and API delivery

## Workflow

1. Inspect routes, controllers, DTO tags, validation, auth middleware, response writers, errors and frontend API wrappers when available. Read base URL/proxy/gateway configuration. Record source revision and mark observed, proposed and unknown behavior separately.
2. Describe the browser request path and response path. Identify public browser APIs versus internal APIs; never expose SQL, Redis or internal RPC directly to the browser. Choose direct HTTP or gateway/BFF based on actual deployment and frontend needs.
3. Define or extract a single authoritative contract. Deliver OpenAPI YAML/JSON plus readable per-endpoint documentation for created or modified endpoints. Reuse existing documentation tools and generated artifacts. In implementation work, store these in the project at its documented locations and link them; in analysis-only work, provide representative documentation inline unless a file is requested. Do not silently refactor the project merely to generate documentation.
4. Check operation IDs, schemas, references, examples and security definitions using existing tooling. Compare registered routes and DTO behavior against the document; verify representative success and failure responses. Do not claim validation or integration passed unless executed.
5. Give frontend setup, type/client generation or manual wrapper conventions, schema-backed mock setup and integration acceptance steps. Use the existing frontend language/framework if known; otherwise give a clearly marked TypeScript example without presuming implementation exists.

## Required per-endpoint documentation

Include purpose and implementation status; HTTP method and full path; environments/base URL; authentication header or cookie and permissions; path/query/header/body parameters; required/optional/default/null/empty semantics; types, formats, limits and enumerations; successful HTTP status and body; every meaningful business and transport error with frontend action; examples of request and actual response. Specify side effects, idempotency, rate limits, uploads/downloads or asynchronous completion where applicable. Unknown behavior must be labeled, not invented.

Define cross-endpoint conventions once: response envelope, business code versus HTTP status, timestamp timezone/format, money currency/precision, pagination with ordering and total semantics, empty list/null handling, trace ID, compatibility/version policy. Represent int64 identifiers safely for JavaScript where relevant; document actual serialization and propose migration if needed. Never leak storage models or raw internal errors into proposed public schemas.

## Browser behavior

- Configure dev proxy or API base URL and production gateway routing. Document same-origin deployment or exact CORS origins/methods/headers; credentialed cross-origin requests require compatible browser/server settings. Add CSRF measures when cookie authentication requires them.
- Follow the server's actual authentication contract, including custom headers in existing projects. For new designs choose a documented token/cookie strategy, expiry, refresh, logout and revocation behavior. Never place secrets in browser code or log tokens.
- Centralize client transport/error mapping. Check HTTP status and application envelope independently when both exist. Handle expired auth, forbidden access, validation, timeout, unavailable service and cancellation explicitly. Prevent concurrent refresh storms if refresh exists.
- Retry only operations whose contract makes replay safe, within deadlines. Disable repeated form submission or use a server-supported idempotency key when required; UI disabling alone is not server-side idempotency.
- Generate types/clients from the agreed OpenAPI contract where suitable; use mocks for parallel development but verify against a real backend before signoff. Avoid treating TypeScript types as runtime validation.

## Feature delivery checklist

For a new administrator-list endpoint, deliver a documented method/path, page/filter defaults and bounds, stable ordering, list item schema, total semantics, operator permissions, empty/success/error examples, frontend list invocation and auth handling, and contract checks. Name the actual files and commands used. Include the endpoint only as a proposal when it is absent from the source.

For the flyfei reference project, preserve observed behavior: public root is /api/mall, admin and customer route groups differ, standard API writer uses a code/msg/err_msg/data envelope with HTTP 200 while auth middleware uses HTTP 401. Determine exact header names from constants before writing an executable request. Captcha has GET generation and POST checking routes; route names mentioning SMS do not prove SMS sending exists. Do not document placeholder authentication as production-ready or list whitelist entries as registered endpoints.
