# Models And Entities

This package contains API-facing models and persistence entities.

- `*Entity` classes represent database storage; plain model classes represent API/domain payloads.
- Many of these files are scaffolded from `src/main/resources/entity-model.yaml`, so update the source model when structural changes are broad.
- Keep JPA annotations, field names, and mapper expectations synchronized.
