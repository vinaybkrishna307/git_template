# Security Policy

This repository is a reference template for secure CI/CD pipeline design, not a
production service — but the policy below reflects how I'd want vulnerability
reporting handled on a real project using this pattern.

## Reporting a Vulnerability

If you find a security issue in a project built on this template:

1. **Do not open a public issue.**
2. Email the maintainer directly (or use GitHub's private vulnerability reporting
   under the Security tab) with a description, reproduction steps, and impact.
3. Expect an acknowledgement within 48 hours and a fix timeline based on severity.

## Scope of automated checks in this template

| Check | Tool | Stage |
|---|---|---|
| Secrets in code | Gitleaks | Every PR |
| Known vulnerable dependencies | Trivy (filesystem) + Dependabot | Every PR / weekly |
| Vulnerable base images / packages in built artifact | Trivy (image scan) | On build |
| Static code vulnerabilities | CodeQL | Manual / scheduled |
| Build tampering | Cosign signing + SLSA provenance attestation | On build |

Severity thresholds (`HIGH,CRITICAL` for filesystem scans, `CRITICAL` for image
scans) are intentionally strict — the pipeline is designed to fail closed rather
than ship a known-vulnerable artifact silently.