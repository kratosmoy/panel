## Why

The service currently mixes fixed security endpoints such as `/api/me` and `/api/auth/*` with the generic entity route `/api/{entity}/**`, which makes user-facing and Grafana-facing traffic share an ambiguous API shape. Now that the dual-chain security model is in place, we need audience-scoped routing so user and Grafana requests can both use generic entity APIs without path conflicts or unclear security boundaries.

## What Changes

- **BREAKING** Replace the legacy generic entity route `/api/{entity}/**` with `/api/user/{entity}/**` for browser user traffic.
- **BREAKING** Expose Grafana generic entity access under `/api/grafana/{entity}/**` instead of sharing the legacy generic route shape.
- Keep fixed security endpoints such as `/api/me`, `/api/auth/login`, and `/api/auth/logout` outside the generic entity routing surface.
- Refactor the generic controller layer so CRUD, query, and metric behavior is shared while user and Grafana routes bind that behavior through separate audience-specific controllers.
- Align Spring Security filter-chain matchers, tests, and deployment docs with the new audience-scoped route layout.

## Capabilities

### New Capabilities
- `audience-scoped-generic-routes`: Expose generic CRUD, query, and metric endpoints under explicit user and Grafana route prefixes, and remove the legacy unscoped entity route.
- `audience-aligned-security-routing`: Keep fixed auth endpoints outside generic routing and align Spring Security path matching with the new user and Grafana API prefixes.

### Modified Capabilities

None.

## Impact

- Affected code: generic controller routing, dynamic entity dispatch, Spring Security filter-chain configuration, and related tests.
- Affected APIs: `/api/user/{entity}/**`, `/api/grafana/{entity}/**`, `/api/me`, `/api/auth/login`, and `/api/auth/logout`.
- Affected systems: browser clients consuming user APIs, Grafana integrations consuming generic entity APIs, and security documentation describing route isolation.
