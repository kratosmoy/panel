## ADDED Requirements

### Requirement: `/api/me` MUST return normalized authorization context for frontend modules
The system SHALL return a normalized backend user context that includes user identity fields and authorization data required for frontend module gating.

#### Scenario: Authenticated user loads profile
- **WHEN** an authenticated user requests `/api/me`
- **THEN** the response contains `id`, `username`, `displayName`, `email`, `groups`, `entitlements`, `permissions`, `dataScopes`, and `claims`

#### Scenario: Upstream claim is absent
- **WHEN** an optional claim needed by the frontend is not present in the PingFederate response
- **THEN** the backend returns a safe default value instead of omitting the user context contract

### Requirement: Authorization lists MUST be derived from mapped PingFederate claims
The system SHALL map configured PingFederate claims into normalized `groups`, `entitlements`, and `permissions` collections for the frontend.

#### Scenario: PingFederate provides group and entitlement claims
- **WHEN** the authenticated user contains configured group and entitlement claims from PingFederate
- **THEN** the backend maps those values into the corresponding normalized arrays in `/api/me`

#### Scenario: Duplicate authorization values are present
- **WHEN** mapped claims contain duplicate group, entitlement, or permission values
- **THEN** the backend returns de-duplicated normalized collections

### Requirement: User authorization context MUST remain distinct from Grafana application identity
The system SHALL expose module authorization context only for authenticated user sessions and SHALL NOT reuse Grafana application principals as frontend user identities.

#### Scenario: Grafana application calls a Grafana endpoint
- **WHEN** a Grafana application principal accesses `/api/grafana/**`
- **THEN** the system treats the caller as an application identity and does not produce browser user context

#### Scenario: Browser user calls `/api/me`
- **WHEN** an authenticated browser user requests `/api/me`
- **THEN** the system returns user-scoped authorization context derived from the user principal rather than any application-client mapping
