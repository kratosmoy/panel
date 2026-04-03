## Context

The current service exposes generic entity APIs through a single dynamic route, `/api/{entity}/**`, while fixed security endpoints such as `/api/me`, `/api/auth/login`, and `/api/auth/logout` live in the same top-level `/api` namespace. After the dual-chain Spring Security work, that shape leaves audience boundaries implicit: browser users and Grafana callers may both need generic entity APIs, but the route layout does not express which identity model owns which path.

This change is cross-cutting because it affects controller mappings, security filter-chain matchers, tests, and the operational security documentation. The new design must preserve the existing dual identity model:
- Browser users authenticate through the OAuth2-backed BFF flow.
- Grafana authenticates as an application client through x509 certificate matching.
- Fixed auth endpoints remain outside the generic entity dispatcher.

## Goals / Non-Goals

**Goals:**
- Replace the legacy `/api/{entity}/**` surface with explicit audience-scoped generic routes.
- Allow both user and Grafana callers to use the same CRUD, query, and metric capabilities without duplicating the underlying generic service logic.
- Keep `/api/me` and `/api/auth/*` as fixed security endpoints that never compete with entity routing.
- Align Spring Security filter-chain matchers and tests with the new route prefixes.

**Non-Goals:**
- Introducing route compatibility aliases for `/api/{entity}/**`.
- Changing the generic service contract, search semantics, or entity registry rules.
- Collapsing user endpoints and Grafana endpoints into one controller class with audience-specific conditionals.
- Changing authentication mechanisms for users or Grafana beyond the route matchers they protect.

## Decisions

### Decision: Split generic routing by audience prefix

We will replace the single dynamic route with two explicit controller entrypoints:
- `/api/user/{entity}/**` for browser user traffic
- `/api/grafana/{entity}/**` for Grafana application traffic

Why this choice:
- The path expresses which identity boundary applies before entity resolution begins.
- Fixed endpoints such as `/api/me` and `/api/auth/*` stay unambiguous.
- Grafana traffic no longer depends on `grafana` accidentally being treated as an entity name.

Alternatives considered:
- Keep `/api/{entity}/**` and special-case reserved names: rejected because it hides security boundaries inside controller logic.
- Use one controller at `/api/{audience}/{entity}/**`: rejected because it centralizes too many audience-specific concerns in one class.

### Decision: Share generic behavior through an abstract audience-aware controller base

CRUD, query, and metric handling will remain shared, but the shared code will live in an abstract base or delegate that accepts the `entity` path variable and resolves the `GenericService` through `EntityRegistry`. Two concrete controllers will bind that shared behavior to their own route prefixes.

Why this choice:
- It preserves one implementation of generic endpoint behavior.
- It keeps user and Grafana routing explicit at the controller level.
- It reduces the chance of user-specific and Grafana-specific behavior drifting over time.

Alternatives considered:
- Duplicate the existing dynamic controller into separate user and Grafana implementations: rejected because it would create parallel code paths for the same generic behavior.
- Force security controllers into the generic hierarchy: rejected because fixed auth endpoints do not model entity operations.

### Decision: Narrow security filter chains to audience-prefixed generic routes

The browser user security chain will protect `/api/user/**` plus the fixed user-facing auth endpoints and OAuth2 callback routes. The Grafana chain will protect `/api/grafana/**` only.

Why this choice:
- The filter chain boundaries now mirror the controller boundaries.
- Grafana requests continue to avoid OAuth2 redirects.
- User requests continue to resolve through session-backed authentication and current-user endpoints.

Alternatives considered:
- Keep the user chain matching `/api/**`: rejected because it would unnecessarily overlap with Grafana generic routes and obscure route ownership.

### Decision: Treat the route split as a breaking API migration

The service will remove `/api/{entity}/**` outright rather than supporting both route shapes.

Why this choice:
- The user explicitly requested a direct cutover.
- Supporting both path shapes would weaken the route-boundary cleanup and add extra test/documentation burden.

Alternatives considered:
- Temporary compatibility redirects or dual mappings: rejected because the goal is to make audience ownership explicit immediately.

## Risks / Trade-offs

- [Existing clients still calling `/api/{entity}/**`] -> Update tests and documentation to make the breaking change visible, and fail fast with 404 for legacy paths.
- [Audience-scoped controllers drift from the shared behavior] -> Keep CRUD, query, and metric implementation in one shared base or delegate.
- [Security matcher edits accidentally widen or narrow access] -> Cover the new routes with focused MockMvc integration tests for both user and Grafana paths.
- [Documentation lags behind code] -> Update `docs/security/dual-chain-spring-security.md` in the same change as the route and matcher edits.

## Migration Plan

1. Add OpenSpec requirements for audience-scoped generic routing and audience-aligned security matching.
2. Write failing tests for the new user and Grafana generic routes, plus legacy route rejection.
3. Refactor the generic controller layer into shared behavior with separate audience-specific controllers.
4. Update security filter-chain matchers to protect `/api/user/**` and `/api/grafana/**` appropriately while preserving `/api/me` and `/api/auth/*`.
5. Update security and controller tests to use the new paths.
6. Update the security deployment notes to describe the new route layout.

Rollback strategy:
- Revert this change and redeploy the previous route layout if downstream callers are not ready for the new paths.
- Because this is a routing and security-matcher change, rollback is code-only and does not require data migration.

## Open Questions

- None for implementation. The audience prefixes, direct cutover, and fixed auth endpoint behavior have been confirmed.
