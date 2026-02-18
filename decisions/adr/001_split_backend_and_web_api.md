# ADR 0001: Split Admin Backend and Public Web API

## Status

Accepted

## Context

The Book Publishing platform currently consists of:

- A core backend service (`book-publishing-backend`) implemented in Kotlin + Spring Boot.
- A native Android administrative application (`book-publishing-app`) used to manage authors, collections, books, and other domain entities.
- OpenAPI contracts published via `book-publishing-api-spec`.

The backend exposes secured REST endpoints protected by JWT and is primarily designed for administrative operations (create, update, delete, and internal management use cases).

A new requirement has emerged:

- Provide a public-facing website to expose the book catalog.
- The website does not require user authentication in its first iteration.
- The public API should be optimized for read workloads.
- Performance and caching capabilities (e.g., Redis) should be supported.
- The administrative backend must remain protected and not publicly exposed.

Additionally, the long-term roadmap includes:

- Stock management
- Shopping cart functionality
- User accounts and authentication
- Potential payment processing

The architectural decision must support current simplicity while enabling future evolution.

## Decision

We will split responsibilities into two independent services:

1. `book-publishing-backend`
    - Core administrative backend.
    - Implements the `ms-catalog` contract.
    - Handles business logic and persistence.
    - Protected by JWT for administrative users.
    - Deployed to Cloud Run with restricted access (not publicly exposed).

2. `book-publishing-web-api`
    - Public web-facing API (BFF – Backend For Frontend).
    - Implements the `api-catalog` contract.
    - Exposes read-only endpoints.
    - Communicates with the backend using secure service-to-service authentication (Cloud Run IAM with identity tokens).
    - Can introduce Redis caching for performance optimization.
    - Deployed as a public Cloud Run service.

Communication between the web API and the backend will use Google Cloud Run IAM-based authentication, granting the web API service account the `run.invoker` role on the backend service.

This design establishes a clear separation between administrative and public concerns.

## Consequences

### Positive

- Clear separation of concerns between administrative and public responsibilities.
- Layered security model:
    - Backend is not publicly accessible.
    - Public web API has limited, read-only responsibilities.
- Enables performance optimization (Redis caching) without impacting the core backend.
- Facilitates independent scaling of read-heavy public traffic.
- Prepares the platform for future features such as:
    - Stock management
    - Shopping cart
    - User authentication
    - Payment processing
- Allows different evolution paths for admin and public APIs.

### Negative

- Increased operational complexity (two services instead of one).
- Additional deployment pipelines and observability requirements.
- Potential additional network latency between services.
- Requires explicit service-to-service authentication setup.

Latency concerns are mitigated by:
- Aggregated endpoints in the web API.
- Redis caching strategies.
- Cloud Run's low-latency internal communication.

## Alternatives Considered

### 1. Modular Monolith with Two "Faces"

A single service exposing:
- Secured administrative endpoints.
- Public read-only endpoints.

Pros:
- Simpler deployment.
- No inter-service communication.
- Reduced operational overhead.

Cons:
- Harder to enforce strict separation of public and admin concerns.
- Increased risk of exposing internal endpoints.
- More difficult to introduce public-facing optimizations (caching, aggregation) cleanly.
- Scaling read-heavy traffic impacts the entire service.

Rejected in favor of clearer separation and future scalability.

---

### 2. API Gateway Directly in Front of the Backend

Use an API Gateway to:
- Route public traffic to specific backend endpoints.
- Keep admin endpoints secured.

Pros:
- Centralized routing and security.
- Built-in rate limiting and quotas.

Cons:
- Does not provide a dedicated BFF layer.
- Public and admin logic still live in the same service.
- Limited flexibility for aggregation and transformation.
- Harder to introduce Redis caching cleanly at the application layer.

Rejected because it does not provide sufficient separation of responsibilities.

---

### 3. CDN Caching Without a Dedicated Web API

Expose read-only endpoints directly from the backend and rely on CDN caching.

Pros:
- Minimal architectural change.
- Reduced application-layer complexity.

Cons:
- Caching limited to HTTP-level responses.
- No aggregation or response shaping.
- Backend remains publicly exposed.
- Harder to evolve toward e-commerce features.

Rejected due to security concerns and lack of long-term flexibility.

## Notes

This decision establishes the foundational architectural pattern for the platform:

- Core domain and administrative logic remain centralized.
- Public-facing concerns are isolated in a dedicated BFF layer.
- Infrastructure-level security (Cloud Run IAM) enforces service boundaries.

Future ADRs may address:
- Introduction of Redis caching strategy.
- API Gateway integration.
- User authentication and authorization for public features.
- Event-driven communication between services.
