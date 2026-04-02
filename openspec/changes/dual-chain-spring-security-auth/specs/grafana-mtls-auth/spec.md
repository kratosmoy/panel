## ADDED Requirements

### Requirement: Grafana application requests MUST authenticate with x509 client certificates
The system SHALL require an x509-authenticated application principal for every request under `/api/grafana/**`.

#### Scenario: Trusted Grafana client accesses a protected endpoint
- **WHEN** a request to `/api/grafana/**` presents a valid client certificate and the certificate maps to a trusted application client
- **THEN** the system authenticates the request as an application principal and continues request processing

#### Scenario: Missing client certificate on Grafana path
- **WHEN** a request to `/api/grafana/**` does not present a client certificate
- **THEN** the system rejects the request with `401 Unauthorized`

### Requirement: Certificate CN MUST map to configured application clients
The system SHALL extract the subject `CN` from the authenticated client certificate and SHALL authenticate the caller only when that `CN` matches a configured application client in the active Spring profile.

#### Scenario: Configured CN matches certificate subject
- **WHEN** the presented client certificate contains subject `CN=grafana-prod` and the active profile contains an application client mapped to `grafana-prod`
- **THEN** the system authenticates the request using that application client definition

#### Scenario: Unknown certificate CN
- **WHEN** the presented client certificate contains a `CN` that is not defined in the active profile application client configuration
- **THEN** the system rejects the request with `401 Unauthorized`

### Requirement: Grafana authentication failures MUST NOT trigger user login redirects
The system SHALL return API-style authentication and authorization errors for `/api/grafana/**` and MUST NOT redirect Grafana callers to PingFederate login pages.

#### Scenario: Authentication fails for Grafana request
- **WHEN** a request to `/api/grafana/**` fails certificate authentication
- **THEN** the response is an HTTP authentication error without a browser login redirect

#### Scenario: Authenticated application lacks required authority
- **WHEN** an authenticated Grafana application principal accesses a Grafana endpoint it is not authorized to use
- **THEN** the system returns `403 Forbidden`
