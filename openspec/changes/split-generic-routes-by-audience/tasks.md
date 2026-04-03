## 1. Test Coverage

- [x] 1.1 Add failing controller and security tests for `/api/user/{entity}/**` and `/api/grafana/{entity}/**`
- [x] 1.2 Add a failing test proving legacy `/api/{entity}/**` generic routes no longer resolve

## 2. Generic Route Refactor

- [x] 2.1 Refactor the generic controller implementation so user and Grafana audience routes share one generic dispatch path
- [x] 2.2 Replace the legacy `/api/{entity}/**` controller mapping with `/api/user/{entity}/**` and `/api/grafana/{entity}/**`

## 3. Security Alignment

- [x] 3.1 Update Spring Security filter-chain matchers to protect `/api/user/**` and `/api/grafana/**` with their respective authentication models
- [x] 3.2 Keep `/api/me` and `/api/auth/*` fixed outside generic entity routing and update integration tests accordingly

## 4. Documentation And Verification

- [x] 4.1 Update `docs/security/dual-chain-spring-security.md` to describe the new audience-scoped generic routes
- [x] 4.2 Run Gradle tests covering controller and security behavior for the new route layout
