# Package Bridge

This directory is still part of the package path rather than a feature boundary.

- Keep code placement intentional; most production classes belong under `core/` or its subpackages.
- Do not spread cross-cutting logic into bridge directories.
- Preserve package-name and folder alignment.
