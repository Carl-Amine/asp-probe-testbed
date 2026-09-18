# asp-probe-testbed

Deliberately broken. One planted violation per ASP control, so a probe run
against this repo should catch something real -- not just prove it doesn't
crash on a healthy repo.

| Control | Planted violation |
|---|---|
| C1 -- branch protection | None configured (default state of a fresh repo) |
| C2 -- Actions permissions minimised | `.github/workflows/build.yml` has no `permissions:` block |
| C3 -- Azure OIDC, no long-lived creds | `.github/workflows/deploy-creds.yml` uses `creds:` instead of `id-token: write`; a secret named `AZURE_CREDENTIALS` exists (placeholder value, not a real credential) |
| C4 -- PR-triggered Azure auth gated | `.github/workflows/deploy-pr-target.yml` runs on `pull_request_target`, calls `azure/login`, declares no `environment:` |
| C5 -- Azure identity scope | Always `MANUAL`; nothing to plant |
| C6 -- admin/workflow restriction | No `CODEOWNERS` file |

Run the probe against this repo (from `ci-review-service`):

```bash
python -m app.probe.cli --repo Carl-Amine/asp-probe-testbed --token <installation-or-PAT-token>
```

Expect FAIL/WARN on C2, C3, C4, C6; MANUAL on C5; FAIL or UNKNOWN on C1
depending on whether the token can read branch protection. If anything
comes back PASS here, the probe or the permissions are wrong -- fix
whichever it is before trusting a clean run on a real repo.

Fork PR smoke test: confirming pull/{n}/head resolves cross-repo.
