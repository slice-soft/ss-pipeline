# ss-pipeline — Reusable GitHub Actions Workflows

A collection of reusable GitHub Actions workflows for automating CI/CD, code analysis, Docker image builds, and release management across all SliceSoft repositories.

---

## Available Workflows

### `ci-go.yml` — Go CI

Runs tests, static analysis, and build for Go projects.

**Inputs**
- `go-version` (string, optional) — Go version to use. Default: `"1.21"`

**Steps:** Go setup with cache → `go mod download` → `go vet` → `go test` with coverage → `go build`

```yaml
jobs:
  ci:
    uses: slice-soft/ss-pipeline/.github/workflows/ci-go.yml@v0
    with:
      go-version: "1.21"
```

---

### `ci-node.yml` — Node.js CI

Runs tests, linting, and build for Node.js projects.

**Inputs**
- `node-version` (string, optional) — Node.js version to use. Default: `"22"`

**Steps:** Node.js setup → smart `node_modules` cache → install → test → lint → build

```yaml
jobs:
  ci:
    uses: slice-soft/ss-pipeline/.github/workflows/ci-node.yml@v0
    with:
      node-version: "22"
```

---

### `build-node.yml` — Node.js Build + Artifact

Checkout, `npm ci`, build, and upload artifact for downstream jobs.

**Inputs**
- `node-version` — Node.js version
- `build-command` — Build command. Default: `npm run build`
- `artifact-name` (required) — Name of the uploaded artifact
- `artifact-path` — Path to upload. Default: `dist/`
- `version` — Injected as `VERSION` env var
- `retention-days` — Artifact retention days

```yaml
jobs:
  build:
    uses: slice-soft/ss-pipeline/.github/workflows/build-node.yml@v0
    with:
      artifact-name: my-dist
      version: ${{ needs.release.outputs.tag_name }}
```

---

### `validate-pr.yml` — PR Label Validation

Validates that PRs have a semver label before merging.

**Required:** at least one of `patch`, `minor`, `major` must be present.

```yaml
jobs:
  validate:
    uses: slice-soft/ss-pipeline/.github/workflows/validate-pr.yml@v0
```

---

### `create-release.yml` — Automated Release

Generates a CHANGELOG from Conventional Commits, creates a version tag, and publishes a GitHub Release using `release-please`.

```yaml
permissions:
  contents: write
  pull-requests: write

jobs:
  release:
    uses: slice-soft/ss-pipeline/.github/workflows/create-release.yml@v0
```

---

### `deploy-cdn-cloudflare.yml` — CDN Deploy to Cloudflare R2

Downloads an artifact and syncs it to a Cloudflare R2 bucket. Supports versioned (`v{version}/`) and `latest/` paths.

**Inputs**
- `artifact-name` (required) — Artifact to download and deploy
- `destination-prefix` — R2 path prefix (e.g. `design-system/`)
- `version` — Version string (with or without `v`)
- `upload-latest` — Also sync to `latest/`. Default: `true`

**Secrets required:** `R2_ACCESS_KEY_ID`, `R2_SECRET_ACCESS_KEY`, `R2_ENDPOINT`, `R2_BUCKET`, `CDN_BASE_URL`

```yaml
jobs:
  deploy:
    uses: slice-soft/ss-pipeline/.github/workflows/deploy-cdn-cloudflare.yml@v0
    with:
      artifact-name: cdn-dist
      destination-prefix: design-system/
      version: ${{ needs.release.outputs.tag_name }}
    secrets: inherit
```

---

### `analyze-code.yml` — Code Analysis

Uses GitHub Linguist to analyze the languages present in the repository and uploads a report artifact.

**Inputs**
- `workdir` (string, optional) — Working directory. Default: `"."`

---

### `build-docker.yml` — Docker Build & Push

Builds and publishes a Docker image to GitHub Container Registry.

**Inputs**
- `workdir` (required) — Working directory
- `dockerfile` (required) — Dockerfile path
- `image_name` (required) — Docker image name
- `version` (required) — Version tag for the image

**Secrets:** `SSH_PRIVATE_KEY` (required for private repo access during build)

---

### `tf-docs.yml` — Terraform Module Docs + PR

Generates Terraform module documentation with `terraform-docs`, injects the output between `<!-- BEGIN_TF_DOCS -->` and `<!-- END_TF_DOCS -->`, and opens a PR against `main` instead of pushing directly.

**Inputs**
- `module-paths` — Newline-separated list of module directories. Default: `.`
- `output-file` — README file name inside each module. Default: `README.md`
- `require-markers` — Fails if an existing README does not contain the TF docs markers. Default: `true`
- `branch` — Optional PR branch name. If omitted, an ephemeral branch is generated.

**Secret required:** `token` with `contents` and `pull-requests` write permissions on the target repo.

```yaml
jobs:
  docs:
    uses: slice-soft/ss-pipeline/.github/workflows/tf-docs.yml@v0
    with:
      module-paths: |
        .
        modules/network
    secrets:
      token: ${{ secrets.GITHUB_TOKEN }}
```

Expected README placeholder in each module:

```md
<!-- BEGIN_TF_DOCS -->
<!-- END_TF_DOCS -->
```

This workflow runs step by step inside the reusable workflow, uses `terraform-docs/gh-actions@v1.4.1` in `inject` mode, and opens a PR only when documentation changes are detected.

---

## Requirements per workflow

| Workflow | Requirement |
|---|---|
| `ci-go.yml` | `go.mod` present, standard Go tests |
| `ci-node.yml` | `package.json` + `package-lock.json` |
| `build-node.yml` | `package.json` + `package-lock.json` |
| `create-release.yml` | Conventional Commits, write permissions |
| `deploy-cdn-cloudflare.yml` | R2 secrets configured, artifact uploaded |
| `build-docker.yml` | Dockerfile, GitHub Container Registry configured |
| `tf-docs.yml` | Terraform module directories, `README.md` marker block for injection-only updates |

---

## Repository Permissions

Ensure your repository has the correct settings under `Settings > Actions > General`:

- Actions permissions: **Allow all actions and reusable workflows**
- Workflow permissions: **Read and write permissions**

---

## Commit Conventions

All SliceSoft repos follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: new feature        → MINOR
fix: bug fix             → PATCH
feat!: breaking change   → MAJOR
docs: documentation
refactor: refactoring
chore: tooling / config
ci: CI/CD changes
```

## Semver PR Labels

The `validate-pr.yml` workflow requires one of:

| Label | Meaning | Version impact |
|---|---|---|
| `patch` | Bug fix or small improvement | 1.0.**x** |
| `minor` | New non-breaking feature | 1.**x**.0 |
| `major` | Breaking change | **x**.0.0 |

---

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md) for repository-specific rules.
The base workflow, commit conventions, and community standards live in [ss-community](https://github.com/slice-soft/ss-community/blob/main/CONTRIBUTING.md).

## Community

| Document | |
|---|---|
| [CONTRIBUTING.md](https://github.com/slice-soft/ss-community/blob/main/CONTRIBUTING.md) | Workflow, commit conventions, and PR guidelines |
| [GOVERNANCE.md](https://github.com/slice-soft/ss-community/blob/main/GOVERNANCE.md) | Decision-making, roles, and release process |
| [CODE_OF_CONDUCT.md](https://github.com/slice-soft/ss-community/blob/main/CODE_OF_CONDUCT.md) | Community standards |
| [SECURITY.md](https://github.com/slice-soft/ss-community/blob/main/SECURITY.md) | How to report vulnerabilities |

---

SliceSoft — Colombia 💙
