# Repository Guide

This repository is a Java 17 / Spring Boot 3.4 backend with Gradle, OpenSpec change records, and local agent tooling.

- Use the Gradle wrapper for project checks: `gradlew.bat test` on Windows.
- Prefer editing durable sources under `src/`, `docs/`, `openspec/`, `gradle/`, `.codex/`, and `.windsurf/`.
- Do not hand-edit generated or local-only directories such as `build/`, `bin/`, `.gradle/`, or `.idea/`.
- `src/main/resources/entity-model.yaml` plus `gradle/templates/` drive scaffold generation for model, mapper, and repository code.
- Security work must keep code, tests, and `docs/security/dual-chain-spring-security.md` aligned.
- OpenSpec change artifacts under `openspec/changes/` should stay consistent with implemented behavior.
