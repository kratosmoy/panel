## Why

The backend currently exposes application APIs without any Spring Security integration, while the frontend monorepo already expects a backend-for-frontend authentication boundary built around `/api/auth/login`, `/api/auth/logout`, and `/api/me`. We need to add a production-ready security model now so the same Spring Boot service can support direct mTLS authentication for Grafana and session-based user authentication through PingFederate without mixing application and user identities.

## What Changes

- Add Spring Security with two isolated `SecurityFilterChain` definitions.
- Introduce a Grafana-only mTLS authentication path under `/api/grafana/**`.
- Authenticate Grafana requests by extracting the client certificate `CN` and matching it against profile-configured `applicationClients`.
- Introduce BFF-style user authentication using Spring Boot as an OAuth2 client with PingFederate authorization code flow.
- Add backend auth endpoints that preserve the frontend monorepo contract for `/api/auth/login`, `/api/auth/logout`, and `/api/me`.
- Build a backend user context model that returns groups, entitlements, permissions, and claims needed for fine-grained module authorization in the frontend.
- Protect existing browser-facing `/api/**` endpoints with authenticated user sessions while keeping Grafana application access isolated under `/api/grafana/**`.
- Add profile-driven security configuration so PingFederate settings, trusted application clients, and development-only relaxations can be managed without source changes.

## Capabilities

### New Capabilities
- `grafana-mtls-auth`: Authenticate Grafana on `/api/grafana/**` by validating the presented x509 client certificate and mapping certificate `CN` values to configured application clients.
- `bff-user-oauth-auth`: Authenticate browser users through PingFederate authorization code login, maintain a backend session, and expose `/api/auth/login`, `/api/auth/logout`, and `/api/me` for the frontend BFF contract.
- `module-authorization-context`: Produce backend user context with groups, entitlements, permissions, and claims so frontend modules can enforce fine-grained access rules.

### Modified Capabilities

None.

## Impact

- Affected code: `build.gradle`, new Spring Security configuration, new auth/controller/config packages, and updates to existing API controller security annotations or request matching.
- Affected APIs: `/api/auth/login`, `/api/auth/logout`, `/api/me`, and new `/api/grafana/**` application endpoints.
- New dependencies: Spring Security, OAuth2 client support, and configuration property binding for auth/application-client settings.
- External systems: PingFederate for user login, client certificate trust material for mTLS, Grafana as an mTLS caller, and the frontend monorepo consuming backend user context for module access.
