# Dual-Chain Security Change

This change captures the repository's Spring Security split between Grafana mTLS and browser OAuth2 session auth.

- Keep `proposal.md`, `design.md`, `tasks.md`, and nested specs aligned with the current implementation.
- Reflect real endpoint contracts such as `/api/auth/login`, `/api/auth/logout`, `/api/me`, and `/api/grafana/**`.
- Update this change set whenever the security design evolves materially.
