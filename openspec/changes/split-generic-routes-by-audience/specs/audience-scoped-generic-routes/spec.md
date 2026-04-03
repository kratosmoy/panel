## ADDED Requirements

### Requirement: User generic entity routes are audience-scoped
The system SHALL expose generic CRUD, query, and metric endpoints for browser user traffic under `/api/user/{entity}/**` and MUST NOT expose those user entity operations through the legacy `/api/{entity}/**` route shape.

#### Scenario: User route resolves generic entity query
- **WHEN** an authenticated browser user sends a generic entity request to `/api/user/{entity}/query`
- **THEN** the system routes the request through the generic entity stack for the addressed entity

#### Scenario: Legacy generic user route is removed
- **WHEN** a caller sends a generic entity request to `/api/{entity}/query`
- **THEN** the system does not resolve that route as a valid generic entity endpoint

### Requirement: Grafana generic entity routes are audience-scoped
The system SHALL expose generic CRUD, query, and metric endpoints for Grafana application traffic under `/api/grafana/{entity}/**` using the same entity registry and generic service behavior as the user-facing generic routes.

#### Scenario: Grafana route resolves generic entity metric request
- **WHEN** an authenticated Grafana caller sends a generic entity request to `/api/grafana/{entity}/metric`
- **THEN** the system routes the request through the generic entity stack for the addressed entity

#### Scenario: Audience prefix is not treated as an entity name
- **WHEN** a caller requests `/api/grafana/{entity}/**`
- **THEN** the system treats `grafana` as the audience prefix and `{entity}` as the entity name
