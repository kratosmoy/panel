# Security Tests

This package verifies auth flows, filter-chain behavior, and security edge cases.

- Cover both browser-user OAuth2 behavior and Grafana certificate-based behavior.
- Keep tests explicit about expected redirects, 401/403 outcomes, and normalized user context.
- Any change to `SecurityConfiguration` or auth controllers should touch this package.
