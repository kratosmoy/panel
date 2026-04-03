# Controllers

This package owns HTTP request mapping and response shaping.

- Keep controllers thin: delegate business logic to services or security collaborators.
- Preserve the audience-scoped generic route conventions, especially `/api/user/{entity}` and `/api/grafana/{entity}`.
- Validation, status codes, and request/response contracts should be explicit and covered by tests.
