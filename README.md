# 🚀 DevOps CI/CD Templates

Centralized, reusable GitHub Actions workflows (`workflow_call`) designed for enterprise GitOps delivery pipelines, high-performance Docker caching, and Kubernetes deployment workflows.

---

## 📦 Available Workflows

### 1. Reusable Docker & GitOps Pipeline (`reusable-docker-gitops.yml`)
Workflow path: `.github/workflows/reusable-docker-gitops.yml`

A complete end-to-end delivery pipeline that:
1. **🧪 Quality Gate (Unit Tests):** Automatically discovers and runs `.sln`/`.slnx` or `.csproj` unit tests before building container images.
2. **🛠️ Docker Buildx & GHA Caching:** Leverages GitHub Actions layer caching (`type=gha,mode=min`) to speed up subsequent builds from minutes to seconds.
3. **🏷️ OCI Tags & Metadata:** Generates immutable commit SHA tags and conditionally updates `latest` tag on the default branch.
4. **🛡️ Security Vulnerability Scan:** Optional Trivy container vulnerability scanner with non-blocking or blocking thresholds.
5. **🔄 GitOps Manifest Synchronization:** Updates Kustomize image tags (`kustomize edit set image ...`) and safely pushes back to Git with `[skip ci]` and `git pull --rebase` conflict protection.
6. **📊 Visual Job Summary:** Generates a structured Markdown summary table in GitHub Actions run overview.

---

## 🛠️ Usage Example

In your application repository (e.g., `inventory-track`), create `.github/workflows/deploy.yaml`:

```yaml
name: CI / CD Pipeline

on:
  push:
    branches:
      - master
      - main

concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

permissions:
  contents: write
  packages: write

jobs:
  deploy:
    name: 🚀 Deploy Application
    permissions:
      contents: write
      packages: write
    uses: yunusekc/devops-ci-templates/.github/workflows/reusable-docker-gitops.yml@v1
    with:
      image_name: yunusekc/inventory-track
      dockerfile_path: ./src/InventoryTrack.WebApi/Dockerfile
      build_context: .
      kustomize_dir: ./src/InventoryTrack.WebApi/k8s
      kustomize_image_name: inventorytrack-image
      run_tests: true
      dotnet_version: '10.0.x'
      test_path: InventoryTrack.slnx
      enable_security_scan: false
    secrets: inherit
```

---

## ⚙️ Parameters (Inputs)

| Parameter | Type | Required | Default | Description |
| :--- | :--- | :--- | :--- | :--- |
| `image_name` | string | **Yes** | - | Target container image name (e.g. `yunusekc/inventory-track`) |
| `dockerfile_path` | string | No | `./Dockerfile` | Relative path to Dockerfile |
| `build_context` | string | No | `.` | Docker build context directory |
| `registry` | string | No | `ghcr.io` | Target container registry URL |
| `kustomize_dir` | string | No | `""` | Directory containing `kustomization.yaml` (skipped if empty) |
| `kustomize_image_name` | string | No | `""` | Image identifier defined in `kustomization.yaml` to bump |
| `run_tests` | boolean | No | `true` | Run unit tests before building container |
| `dotnet_version` | string | No | `'10.0.x'` | .NET SDK version to install for test runner |
| `test_path` | string | No | `""` | Target `.sln`, `.slnx` or `.csproj`. Auto-detects if omitted |
| `enable_security_scan` | boolean | No | `false` | Run Trivy container vulnerability scan |

---

## 🔒 Required Secrets

| Secret | Required | Description |
| :--- | :--- | :--- |
| `ghcr_token` | No | Container registry access token. Defaults to `${{ secrets.GITHUB_TOKEN }}` if omitted. |

---

## 🏷️ Versioning Strategy

- `@v1` (Major release tag - recommended for callers to receive backwards-compatible updates)
- `@v1.0.0` (Semantic pinned tag)
- `@main` (Bleeding edge development branch)
