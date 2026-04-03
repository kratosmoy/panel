# Main Resources

This folder contains application configuration, schema/data seeds, and scaffold inputs.

- Keep profile files (`application*.properties`) free of secrets; use environment variables for deploy-specific values.
- Update schema and seed data deliberately so local and test startup remain deterministic.
- Treat `entity-model.yaml` as a generator input that should stay consistent with model, mapper, and repository code.
