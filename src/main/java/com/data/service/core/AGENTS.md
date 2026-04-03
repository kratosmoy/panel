# Core Application Package

This is the Spring Boot application root for runtime code.

- `CoreApplication` boots the service; keep package-wide conventions centered here.
- Push HTTP, persistence, search, and security details into their dedicated subpackages.
- Changes that cross package boundaries should usually update resources and tests in the same workstream.
