## ADDED Requirements

### Requirement: Fixed auth endpoints stay outside generic entity routing
The system SHALL keep fixed security endpoints such as `/api/me`, `/api/auth/login`, and `/api/auth/logout` outside the generic entity routing surface so they are never resolved as entity names.

#### Scenario: Current user endpoint remains fixed
- **WHEN** an authenticated browser user requests `/api/me`
- **THEN** the system returns the current user context instead of attempting generic entity resolution

#### Scenario: Auth entrypoint remains fixed
- **WHEN** a caller requests `/api/auth/login`
- **THEN** the system processes the login entrypoint instead of attempting generic entity resolution

### Requirement: Security filter chains align with audience-prefixed routes
The system MUST apply the browser user security chain to `/api/user/**` and the fixed user auth endpoints, and MUST apply the Grafana security chain to `/api/grafana/**`.

#### Scenario: User generic route requires browser authentication
- **WHEN** an unauthenticated browser caller requests `/api/user/{entity}`
- **THEN** the system rejects the request according to the browser user security chain

#### Scenario: Grafana generic route requires application authentication
- **WHEN** a caller without a trusted client certificate requests `/api/grafana/{entity}`
- **THEN** the system rejects the request according to the Grafana security chain without redirecting to OAuth2 login
