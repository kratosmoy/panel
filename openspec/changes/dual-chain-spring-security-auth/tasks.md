## 1. Security foundation

- [x] 1.1 Add Spring Security and OAuth2 client dependencies to `build.gradle`
- [x] 1.2 Create configuration properties for PingFederate client settings, claim mapping, and profile-driven `applicationClients`
- [x] 1.3 Add profile-specific application configuration for local, test, and target environments, including TLS client-auth posture and dev-only H2 console allowances

## 2. Grafana mTLS chain

- [x] 2.1 Implement the `/api/grafana/**` `SecurityFilterChain` with API-style `401/403` handling and no OAuth2 redirects
- [x] 2.2 Implement x509 principal extraction that reads certificate subject `CN` and resolves it against configured `applicationClients`
- [x] 2.3 Introduce application principal and authority mapping types for trusted Grafana callers

## 3. User BFF OAuth2 chain

- [x] 3.1 Implement the browser-facing `SecurityFilterChain` for `/api/**`, `/oauth2/**`, and `/login/oauth2/**`
- [x] 3.2 Configure PingFederate OAuth2 client registration and authorization code login for backend-managed sessions
- [x] 3.3 Implement `/api/auth/login` return-url handling and successful post-login redirect behavior
- [x] 3.4 Implement logout handling for `/api/auth/logout` with session invalidation and a clear extension point for provider logout

## 4. User context and module authorization

- [x] 4.1 Implement a claim-mapping component that normalizes PingFederate claims into groups, entitlements, permissions, data scopes, and claims
- [x] 4.2 Implement `/api/me` to return the frontend-compatible backend user context contract
- [x] 4.3 Protect existing browser-facing `/api/{entity}` endpoints so they require authenticated user sessions

## 5. Verification and rollout readiness

- [x] 5.1 Add tests for Grafana x509 authentication success, missing certificate rejection, and unknown `CN` rejection
- [x] 5.2 Add tests for user login entrypoint behavior, authenticated session access, `/api/me`, and logout
- [x] 5.3 Add tests proving Grafana requests never redirect to PingFederate and user requests do not resolve as application principals
- [x] 5.4 Document deployment constraints for shared-listener mTLS, including the future option of a dedicated Grafana listener if strict transport isolation is required
