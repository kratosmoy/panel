# Grafana mTLS Spec

This capability covers application authentication for Grafana endpoints.

- Keep the spec centered on `/api/grafana/**`, x509 client certificates, and CN-based client mapping.
- Preserve the distinction between transport realities and application-layer enforcement.
- Update the spec when certificate matching, authorities, or failure behavior changes.
