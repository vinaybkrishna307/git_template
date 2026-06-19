# git_template

# Secure CI/CD Pipeline — Reference Template

A reference architecture for a security-first CI/CD pipeline that I built to formalize
supply-chain security practices (SAST, secrets scanning, SBOM, image signing, build
provenance) into something reusable. The intent is to drop this into a real project on
day one and wire up the placeholders to the target stack.

**This is a template, not a working app.** There's no application code in this repo on
purpose — see "What's intentionally left out" below.

## Pipeline overview

| Workflow | Trigger | What it does |
|---|---|---|
| `cr-pr.yml` | PR → `main` | Tests, lint, **Gitleaks secrets scan**, Trivy filesystem scan → SARIF uploaded to GitHub Security tab, auto-comments on failure |
| `ci-template.yaml` | Push → `main` | Docker build → push to GHCR → Trivy image scan → SBOM (SPDX) → upload to Dependency-Track → Cosign image signing → SLSA provenance attestation → deploy to dev |
| `ci-staging.yaml` | On successful CI run | Deploy to staging → smoke test → bake period → health check → rollback on failure |
| `ci-prod.yaml` | Manual dispatch | Manual approval gate → canary rollout → health check → rollback on failure |
| `code-scan.yml` | Manual dispatch | CodeQL static analysis |

## Security decisions worth calling out

- **Least-privilege tokens everywhere.** Every workflow declares an explicit, narrow
  `permissions:` block (e.g. `contents: read`, `id-token: write` only where OIDC is
  needed) instead of relying on the default broad `GITHUB_TOKEN` scope.
- **OIDC over long-lived credentials.** `id-token: write` is used for keyless signing
  (Cosign) — the same pattern extends cleanly to OIDC-based cloud auth (e.g. AWS) so
  no static cloud secrets need to live in GitHub.
- **Shift-left, not just shift-right.** Secrets and dependency scanning happen at the
  PR stage (before merge), not just at deploy time.
- **Supply chain over just vulnerability scanning.** SBOM generation + Cosign signing
    + SLSA provenance attestation means a deployed image's full build history can be
      verified, not just "did Trivy find a CVE."
- **CODEOWNERS on `.github/workflows/*`** — changes to CI/CD config itself require
  review, since the pipeline is as much an attack surface as the app code.

## What's intentionally left out (and the plan for it)

- [ ] Terraform for the infra this assumes — GitHub OIDC IAM role, GHCR/ECR registry,
  cluster namespace. Next addition.
- [ ] Real Kubernetes manifests / Helm chart for the `kubectl set image` step.
- [ ] Prometheus/Grafana config backing the "health check" steps.
- [ ] A minimal sample app + Dockerfile to make the pipeline runnable end-to-end.

## Tools used and why

| Tool | Purpose |
|---|---|
| Gitleaks | Secrets detection on every PR, before code reaches a registry |
| Trivy | Filesystem scan (PR) + container image scan (build) |
| CodeQL | Static application security testing |
| Anchore SBOM Action | SPDX SBOM generation per image |
| Dependency-Track | SBOM ingestion for ongoing component vulnerability tracking |
| Cosign | Keyless container image signing |
| SLSA provenance attestation | Tamper-evident build provenance |