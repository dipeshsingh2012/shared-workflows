# Shared GitHub Actions Workflows (`shared-workflows`)

Central repository hosting reusable GitHub Actions workflows for **Hiljhil Roasters / MyCommerce** micro-frontends (MFEs), shared UI libraries, and backend microservices.

---

## ⚡ Highlights

* **100% Keyless GCP Deployment**: Uses **Workload Identity Federation (WIF)**. Repositories in your account require **ZERO secrets** to deploy to Google Cloud Storage.
* **Instant Onboarding**: Any new or existing MFE repository needs only a ~15 line workflow file.
* **Smart Cache-Control**: Automatically invalidates `remoteEntry.js` (`no-cache, no-store, must-revalidate`) while serving immutable hashed chunks with 1-year caching.

---

## 📦 Available Workflows

| Workflow | File Path | Use Case |
| :--- | :--- | :--- |
| **Deploy to Cloud Run** | [`.github/workflows/deploy-cloud-run.yml`](.github/workflows/deploy-cloud-run.yml) | Builds FastAPI/Python Docker containers, keylessly authenticates to GCP via WIF, pushes to Artifact Registry, and deploys to Google Cloud Run with live health checks. |
| **Database Migration & Seed** | [`.github/workflows/db-migration.yml`](.github/workflows/db-migration.yml) | Runs Alembic schema migrations and executes seed scripts against Neon / Cloud SQL PostgreSQL using repository secrets. |
| **Deploy MFE to GCS** | [`.github/workflows/deploy-mfe-gcs.yml`](.github/workflows/deploy-mfe-gcs.yml) | Builds Vite/Webpack/Next.js MFEs, keylessly authenticates to GCP via WIF, syncs bundle to GCS bucket (`gs://mycommerce/`), and applies strict `no-cache` headers on `remoteEntry.js`. |
| **MFE CI Validation** | [`.github/workflows/mfe-ci.yml`](.github/workflows/mfe-ci.yml) | Installs dependencies, runs TypeScript type-checking (`tsc --noEmit`), and verifies production builds for Pull Requests. |

---

## 🚀 Quickstart: How to Use in Any MFE Repository

### 1. Keyless MFE Deployment to Google Cloud Storage

Add this file to your MFE repository at `.github/workflows/deploy.yml`:

```yaml
name: Deploy MFE to GCS

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  id-token: write    # Required for keyless Workload Identity Federation

jobs:
  deploy:
    uses: dipeshsingh2012/shared-workflows/.github/workflows/deploy-mfe-gcs.yml@main
    with:
      dest_dir: 'mfes/<your-mfe-name>'   # e.g. mfes/testimonials-ui, mfes/counter-check, mfes/cart-ui
```

> [!NOTE]
> **No Secrets Required**: Because Workload Identity Federation is pre-configured at the account level (`attribute.repository_owner == 'dipeshsingh2012'`), you do **not** need to add any secrets under GitHub repository settings!

---

### 2. Reusable MFE Pull Request CI Check

Add this file to your MFE repository at `.github/workflows/ci.yml`:

```yaml
name: CI Verification

on:
  pull_request:
    branches:
      - main

jobs:
  verify:
    uses: dipeshsingh2012/shared-workflows/.github/workflows/mfe-ci.yml@main
    with:
      run_typecheck: true
      build_command: 'npm run build'
```

---

## ⚙️ Workflow Inputs Reference

### `deploy-mfe-gcs.yml`

#### Inputs (`with:`)
| Name | Required | Default | Description |
| :--- | :---: | :--- | :--- |
| `dest_dir` | **Yes** | — | Target path within the bucket (e.g. `mfes/testimonials-ui`). |
| `bucket` | No | `mycommerce` | Target Google Cloud Storage bucket name. |
| `node_version` | No | `20` | Node.js runtime version. |
| `build_command` | No | `npm run build` | Command executed to produce the production build. |
| `dist_dir` | No | `dist` | Local directory containing build artifacts. |
| `cache_control_remote_entry` | No | `no-cache, no-store, must-revalidate` | Cache-Control header applied to `remoteEntry.js`. |
| `cache_control_html` | No | `no-cache, no-store, must-revalidate` | Cache-Control header applied to `index.html`. |
| `workload_identity_provider` | No | `projects/518971663061/.../providers/github-provider` | GCP Workload Identity Provider resource URI. |
| `service_account` | No | `gh-actions-deployer@mycommerce-508208.iam.gserviceaccount.com` | Service account to impersonate. |

---

## 🔒 Security Architecture (Workload Identity Federation)

* **GCP Project**: `mycommerce-508208`
* **GCP Project Number**: `518971663061`
* **Workload Identity Pool**: `github-actions-pool`
* **Provider**: `github-provider` (`https://token.actions.githubusercontent.com`)
* **Policy Binding**: Any repository owned by `dipeshsingh2012` is authorized to impersonate `gh-actions-deployer@mycommerce-508208.iam.gserviceaccount.com`.
* **Zero Long-Lived Keys**: Tokens are ephemeral OIDC assertions minted by GitHub and exchanged via Google STS, expiring automatically after workflow completion.
