## Context

`panel` is a Spring Boot 3.4 service that currently exposes `/api/**` without Spring Security. The frontend monorepo is already implemented as a backend-for-frontend client: it redirects the browser to `/api/auth/login`, expects backend-managed session authentication, and loads the current user via `/api/me`. We also need to support Grafana as an application caller using mTLS with x509 client certificates, where certificate subject `CN` values are matched against profile-configured `applicationClients`.

This change is security-sensitive and cross-cutting because it introduces:
- A new security framework and dependencies.
- Two authentication modes with different trust boundaries.
- External PingFederate integration for user login.
- A new authorization context contract consumed by frontend modules.

The current service structure is simple: a dynamic REST controller serves browser-facing APIs under `/api/{entity}`. That simplicity is helpful because we can add clear routing boundaries without refactoring business logic first.

## Goals / Non-Goals

**Goals:**
- Add a dual-chain Spring Security model that isolates Grafana application authentication from browser user authentication.
- Use Spring Boot as an OAuth2 client for PingFederate authorization code login.
- Preserve the frontend BFF contract for `/api/auth/login`, `/api/auth/logout`, and `/api/me`.
- Support fine-grained frontend module access by returning normalized groups, entitlements, permissions, and claims from `/api/me`.
- Authenticate Grafana on `/api/grafana/**` by matching certificate `CN` values to configured `applicationClients`.
- Keep environment-specific security settings in Spring profiles and external configuration.

**Non-Goals:**
- Splitting the service into separate deployables for user and application traffic.
- Implementing direct browser-to-resource-server token flows.
- Defining Grafana dashboard authorization semantics beyond recognizing the calling application.
- Reworking current business APIs into resource-server bearer-token endpoints.
- Introducing a centralized policy engine in this change.

## Decisions

### Decision: Use two ordered `SecurityFilterChain` definitions

We will implement two explicit chains:
- Chain 1: `/api/grafana/**` for Grafana mTLS.
- Chain 2: `/api/**`, `/oauth2/**`, and `/login/oauth2/**` for user BFF flows and authenticated browser APIs.

Why this choice:
- It isolates application identity from user identity.
- It avoids accidental OAuth2 redirects for Grafana callers.
- It aligns with the frontend monorepo, which already assumes a session-based BFF.

Alternatives considered:
- Single shared chain with both `x509()` and `oauth2Login()`: rejected because failure handling and identity semantics become ambiguous.
- Split services: rejected for now because it adds deployment and routing overhead beyond the current scope.

### Decision: Use path isolation for identity boundaries, but document TLS limitations

Grafana endpoints will live under `/api/grafana/**` and never share the current `/api/{entity}` browser API surface.

Important constraint:
- TLS client-certificate negotiation happens before the request path is known.
- On a single listener, strict transport-level mTLS for only `/api/grafana/**` is not possible.

Implementation choice:
- The initial implementation will use a shared HTTPS listener with optional client certificates (`client-auth=want`) so browser user traffic can still connect without a certificate.
- The Grafana chain will enforce x509 authentication at the application layer by requiring a resolved certificate-backed principal on `/api/grafana/**`.

Alternatives considered:
- `client-auth=need` on the shared listener: rejected because ordinary browser users would be unable to complete the PingFederate flow without presenting a client certificate.
- Dedicated port/host for Grafana: deferred, but documented as the upgrade path if strict transport-level mTLS isolation becomes mandatory.

### Decision: Represent trusted Grafana callers as profile-configured application clients

We will add configuration properties similar to:
- `security.application-clients[*].name`
- `security.application-clients[*].certificate-cn`
- `security.application-clients[*].authorities`
- `security.application-clients[*].claims`

At runtime, the Grafana x509 authenticator will:
1. Read the client certificate from `jakarta.servlet.request.X509Certificate`.
2. Extract the certificate subject `CN`.
3. Match the `CN` against configured application clients.
4. Create an application principal with application-specific authorities.

Why this choice:
- It satisfies the requirement that trusted application callers come from Spring profile configuration.
- It keeps trust configuration externalized and auditable.
- It avoids coupling Grafana auth to user directories or OAuth2 claims.

Alternatives considered:
- Hardcoded `CN` allowlist in Java: rejected because it is not profile-driven.
- Full certificate fingerprint mapping: deferred because the user requirement is `CN`-based matching.

### Decision: Use Spring OAuth2 Client with PingFederate for user login

Spring Boot will act as the OAuth2 client using PingFederate registration properties in profile-specific config. The user flow will use:
- `GET /api/auth/login?returnUrl=...` to start login.
- Spring OAuth2 authorization redirect to PingFederate.
- Spring callback handling on `/login/oauth2/code/{registrationId}`.
- Session creation after successful login.
- `GET /api/me` to expose the normalized user context expected by the frontend.
- `GET /api/auth/logout` or `POST /logout` integration for local session termination.

Why this choice:
- It matches the frontend contract already implemented in `frontend-monorepo`.
- It keeps tokens and code exchange on the backend.
- It allows user-facing APIs to continue using browser cookies instead of bearer tokens.

Alternatives considered:
- Frontend-managed PKCE and bearer tokens: rejected because the frontend explicitly does not call resource servers directly.
- Custom hand-rolled OAuth2 code exchange: rejected because Spring Security already provides the correct primitives.

### Decision: Build a normalized backend user context for module authorization

The frontend expects `/api/me` to return a user context with:
- `id`
- `username`
- `displayName`
- `email`
- `groups`
- `entitlements`
- `permissions`
- `dataScopes`
- `claims`

We will normalize PingFederate claims into that contract through a dedicated mapper component with profile-configured claim names. The mapper will also flatten role-like values into consistent arrays to support frontend module access checks.

Why this choice:
- It preserves the frontend contract and avoids frontend changes.
- It isolates PingFederate claim variability behind backend mapping logic.
- It gives the backend a stable place to enforce default values and validation.

Alternatives considered:
- Return raw PingFederate claims from `/api/me`: rejected because it leaks provider-specific shape into the frontend.
- Encode module authorization rules directly in the backend for every route: deferred because the current requirement is to support frontend module-level gating with normalized claims.

### Decision: Keep Grafana and user authorization semantics separate

Grafana principals will receive authorities prefixed for application identity, for example `ROLE_APP_GRAFANA` and optional application scopes from configuration.

User principals will receive authorities derived from mapped claims, for example:
- `GROUP_<name>`
- `ENTITLEMENT_<name>`
- `PERMISSION_<name>`

Why this choice:
- It prevents application clients from accidentally passing user authorization checks.
- It keeps future method-security rules understandable.

Alternatives considered:
- Sharing one flat authority namespace: rejected because collisions between app and user identities would be hard to reason about.

### Decision: Preserve development ergonomics explicitly

The service currently exposes the H2 console. During development we will allow `/h2-console/**` only under a development profile and relax frame options for that profile only.

Why this choice:
- It avoids breaking local development immediately.
- It keeps production defaults secure.

Alternatives considered:
- Global permit-all for H2 console: rejected because it weakens production defaults.

## Risks / Trade-offs

- [Shared listener with `client-auth=want` may request certificates on non-Grafana connections] -> Document this behavior, keep Grafana traffic on a dedicated hostname where possible, and preserve a future path to a dedicated mTLS listener if browser UX becomes problematic.
- [PingFederate claim names may not match frontend expectations] -> Make claim mapping property-driven and add contract tests for `/api/me`.
- [Existing `/api/**` endpoints will become authenticated and may break tests] -> Add focused MVC and integration tests, then update tests incrementally under the new security model.
- [Grafana certificate `CN` matching is less strict than full certificate pinning] -> Keep the trust store authoritative for certificate validation and scope CN matching to a small configured allowlist.
- [Dual identity types increase code paths and failure modes] -> Use distinct principal types, distinct chains, and explicit failure handlers.
- [Session-based user auth introduces CSRF concerns] -> Keep CSRF enabled by default for browser-facing flows and only carve out explicit exceptions when required by actual endpoint behavior.

## Migration Plan

1. Add Spring Security and OAuth2 client dependencies and property classes.
2. Introduce dual filter chains with path boundaries, dev-only H2 allowances, and separate failure behavior.
3. Implement Grafana x509 principal resolution from configured `applicationClients`.
4. Implement PingFederate OAuth2 client registration, login entrypoint, success handling, logout handling, and `/api/me`.
5. Add tests for Grafana mTLS resolution, unauthenticated browser access, successful user login/session behavior, and `/api/me` normalization.
6. Roll out first in a non-production profile with PingFederate test credentials and trusted Grafana certificates.
7. If certificate prompting or ingress behavior is problematic, move Grafana traffic to a dedicated connector or ingress route before production rollout.

Rollback strategy:
- Disable the new security profile and redeploy the previous unsecured configuration if integration fails in lower environments.
- Because this change adds dependencies and routing but does not alter persistence, rollback is operational rather than data-migratory.

## Open Questions

- Which exact PingFederate claims should map to `groups`, `entitlements`, `permissions`, and `dataScopes` in `/api/me`?
- Does the deployment environment terminate TLS directly in Spring Boot, or is there an ingress/proxy that could affect client-certificate propagation?
- Should `/api/auth/logout` only invalidate the local session, or should it also trigger PingFederate RP-initiated logout when supported?
- Do any existing `/api/**` endpoints need application-level access for Grafana outside the new `/api/grafana/**` namespace?
