# Generator Templates

These Groovy templates generate model, entity, mapper, repository, and service scaffolding.

- Treat `src/main/resources/entity-model.yaml` as the source of truth for entity shape.
- Avoid baking entity-specific business rules into shared templates unless all generated classes need them.
- If a template changes, regenerate and review downstream Java files for drift.
