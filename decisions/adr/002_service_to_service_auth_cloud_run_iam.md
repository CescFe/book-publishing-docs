# ADR 002: Service-to-Service Authentication using Cloud Run IAM

## Status

Accepted

## Context

As established in ADR 001, the platform is split into:

- `book-publishing-backend` (core administrative backend)
- `book-publishing-web-api` (public web-facing BFF)

The backend must not be publicly accessible and should only accept:

- Authenticated administrative requests (JWT-based authentication)
- Authenticated internal requests from the web API (service-to-service communication)

Because both services are deployed in Google Cloud Run, a secure and cloud-native authentication mechanism is required to:

- Prevent unauthorized invocation of the backend
- Avoid exposing internal endpoints publicly
- Eliminate shared secrets or static API keys
- Maintain a clean separation of concerns between infrastructure-level and application-level security

The solution must align with Google Cloud best practices and support future scalability.

## Decision

We will use Google Cloud Run IAM-based service-to-service authentication.

### Key Elements of the Decision

1. Backend Access Control
    - The `book-publishing-backend` service will NOT allow unauthenticated invocations.
    - Only specific identities will be allowed to invoke the service.
    - The `roles/run.invoker` IAM role will be granted exclusively to:
        - The service account used by `book-publishing-web-api`
        - (Optionally) administrative identities if needed

2. Identity-Based Invocation
    - `book-publishing-web-api` will run using a dedicated Service Account.
    - When calling the backend, it will generate a Google-signed Identity Token (OIDC).
    - The token audience (`aud`) will match the backend service URL.
    - The token will be sent in the `Authorization: Bearer <ID_TOKEN>` header.

3. Infrastructure-Level Enforcement
    - Cloud Run IAM will validate identity before the request reaches the backend container.
    - Unauthorized requests will be rejected by the platform.

4. Ingress Configuration
    - The backend will be configured with restricted ingress:
        - Preferably "Internal" or
        - "Internal and Cloud Load Balancing" if required.
    - The backend will not be exposed publicly.

5. Separation of Security Concerns
    - Administrative endpoints remain protected with JWT (application-level authentication).
    - Service-to-service calls are protected with IAM (infrastructure-level authentication).

## Consequences

### Positive

- Strong identity-based authentication without shared secrets.
- No need to manage API keys between services.
- Backend is not publicly invokable.
- Follows Google Cloud best practices.
- Minimal application-layer complexity.
- Clear separation between:
    - User authentication (JWT)
    - Service authentication (IAM identity)

### Negative

- Requires IAM configuration and service account management.
- Slightly more complex local development setup.
- Developers must understand identity token generation.
- Adds cloud-provider coupling (Cloud Run specific).

## Alternatives Considered

### 1. Static API Key or Shared Secret

The web API would send a static API key to the backend.

Pros:
- Simple to implement.
- Works in any infrastructure.

Cons:
- Secret rotation complexity.
- Risk of accidental leakage.
- No identity-based access control.
- Not aligned with cloud-native security practices.

Rejected due to security and operational concerns.

---

### 2. mTLS Between Services

Use mutual TLS certificates between services.

Pros:
- Strong transport-level authentication.
- Infrastructure-agnostic.

Cons:
- Certificate lifecycle management complexity.
- Overkill for Cloud Run managed environment.
- More operational overhead.

Rejected in favor of managed IAM-based identity.

---

### 3. Exposing Backend Publicly with JWT Only

Allow backend to be publicly accessible and rely solely on JWT validation.

Pros:
- Simpler networking.
- No need for IAM-based service authentication.

Cons:
- Backend becomes publicly reachable.
- Increased attack surface.
- Harder to enforce internal-only policies.

Rejected due to security concerns.

## Notes

This ADR complements ADR 001 and formalizes the trust boundary between services.

Authentication layers are now clearly defined:

- User → Web API (public, no login initially)
- Web API → Backend (IAM identity token)
- Admin App → Backend (JWT authentication)

Future ADRs may define:

- API Gateway integration
- Redis caching strategy
- Public user authentication (OAuth2 / OpenID Connect)
- Role-based authorization for storefront users
