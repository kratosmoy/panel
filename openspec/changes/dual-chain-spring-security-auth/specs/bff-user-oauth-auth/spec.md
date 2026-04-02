## ADDED Requirements

### Requirement: Browser users MUST authenticate through backend-managed OAuth2 login
The system SHALL initiate PingFederate authorization code login from the backend and SHALL maintain authenticated browser access through a backend session.

#### Scenario: Frontend starts user login
- **WHEN** the browser navigates to `/api/auth/login?returnUrl=%2Fworkspace`
- **THEN** the backend starts the PingFederate authorization code flow and preserves `/workspace` as the post-login return destination

#### Scenario: PingFederate login completes successfully
- **WHEN** PingFederate redirects back with a successful authorization code response
- **THEN** the backend exchanges the code, creates an authenticated user session, and redirects the browser to the validated return URL

### Requirement: Browser-facing APIs MUST require authenticated user sessions
The system SHALL require an authenticated user session for browser-facing `/api/**` endpoints other than explicitly permitted auth and development endpoints.

#### Scenario: Unauthenticated user requests `/api/me`
- **WHEN** a browser without an authenticated session requests `/api/me`
- **THEN** the system returns `401 Unauthorized`

#### Scenario: Authenticated user requests a protected API
- **WHEN** a browser with an authenticated user session requests a protected `/api/**` endpoint
- **THEN** the system authorizes the request using the user session without requiring a bearer token

### Requirement: Backend auth endpoints MUST preserve the frontend BFF contract
The system SHALL expose backend auth endpoints compatible with the frontend monorepo contract for login, logout, and current-user lookups.

#### Scenario: Frontend signs the user out
- **WHEN** the browser requests `/api/auth/logout`
- **THEN** the backend clears the authenticated user session before completing logout

#### Scenario: Frontend loads current user context
- **WHEN** the browser requests `/api/me` with an authenticated session
- **THEN** the backend returns the normalized current-user response expected by the frontend
