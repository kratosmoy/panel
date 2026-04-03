# Main Sources

This directory contains production Java code and runtime resources.

- Keep Java sources under `java/` and environment-neutral resources under `resources/`.
- When changing API behavior, review both code and profile-backed configuration.
- Generated model scaffolding depends on resources and Gradle templates; keep those inputs aligned.
