# Security

This package owns authentication, authorization context mapping, and Spring Security configuration.

- Preserve the separation between Grafana application auth and browser user auth.
- Changes here should be reflected in security tests and `docs/security/dual-chain-spring-security.md`.
- Be careful with filter-chain matchers, entrypoints, claim mapping, and session behavior; small edits can affect the whole API surface.
