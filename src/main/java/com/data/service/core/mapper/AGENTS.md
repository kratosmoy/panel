# Mappers

This package maps domain models to JPA entities and back.

- Treat `*MapperBase` classes as generator-owned unless you are intentionally changing the scaffold output.
- Prefer custom hand-written behavior in the non-base mapper classes.
- Keep mapping logic deterministic and consistent with `entity-model.yaml`.
