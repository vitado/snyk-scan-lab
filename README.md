# Snyk Scan Lab

A parameterized stand for manually running Snyk scans against **arbitrary** public repositories or images. Results are downloaded as a run artifact (`snyk.json` + `snyk.sarif`). Suitable for re-scans: the same workflow with different parameters.

## Prerequisites

Add the `SNYK_TOKEN` secret to the repository (free Snyk still requires authentication):

```bash
gh secret set SNYK_TOKEN -R vitado/snyk-scan-lab
```

The token can be obtained at [app.snyk.io/account](https://app.snyk.io/account) (General → API Token).

## How to run

**Via the UI:**  
Actions → "Snyk scan" → Run workflow → select `type` / fill in `target` / select `severity`.

**Via the CLI:**

```bash
# SCA scan of a repository
gh workflow run snyk-scan.yml \
  -f type=sca \
  -f target=epam/ai-dial-chat \
  -f severity=high

# SAST scan of a specific branch
gh workflow run snyk-scan.yml \
  -f type=code \
  -f target=epam/ai-dial-core \
  -f ref=main \
  -f severity=medium

# Container scan of an image
gh workflow run snyk-scan.yml \
  -f type=container \
  -f target=alpine:3.19 \
  -f severity=critical
```

## Parameters

| Parameter  | Values                               | Default      | Description |
|------------|--------------------------------------|--------------|----------|
| `type`     | `sca` / `code` / `container`         | `sca`        | Scan type |
| `target`   | string, required                     | —            | **owner/repo** for sca/code; **image:tag** for container |
| `ref`      | branch/tag/SHA                       | `""` (default branch) | Git ref for sca/code |
| `severity` | `low` / `medium` / `high` / `critical` | `low`      | Minimum severity level to report |

> **Important:** for the `sca` and `code` types the `target` parameter must be a slug of the form `owner/repo`
> (for example `epam/ai-dial-chat`), because `actions/checkout` is used.  
> For the `container` type — a full image ref (`alpine:3.19`, `ghcr.io/org/image:tag`).

## Where to find the results

- **Run → Artifacts → `snyk-results-<type>`** — an archive with `snyk.html`, `snyk.json`,
  `snyk.sarif` (for sca/container; `code` only has SARIF). Open the `.html` in a browser —
  a ready-made report; `.json` is machine-readable; `.sarif` is for importing into other tools.
- **The scan step log** in the run — Snyk's text output with the issues found.

We deliberately do **not** publish to Security → Code scanning: the stand scans arbitrary
third-party targets, their vulnerabilities are not related to this repo and on the Security tab
would only cause confusion. Code scanning is appropriate when the repo = what you are scanning.

## Limitations (free-tier Snyk)

- `code` (SAST) requires **Snyk Code** enabled in the organization settings
  (app.snyk.io → Settings → Snyk Code). Otherwise — `SNYK-CODE-0005 / 403`.
- There is a monthly test limit on the free plan.

## Note

This repository is a *scanning tool*, not an object of scanning. It is not required to contain the scanned code.
