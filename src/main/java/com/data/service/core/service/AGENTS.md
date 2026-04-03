# Services

This package contains business and data orchestration logic.

- Keep persistence composition, query handling, and metric calculations here rather than in controllers.
- Preserve the generic service abstractions unless there is a strong reason to specialize.
- When metric or query behavior changes, update the corresponding search and service tests.
