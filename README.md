# Shared GitHub Actions Workflows (`shared-workflows`)

Central repository hosting reusable GitHub Actions workflows for **Hiljhil Roasters / MyCommerce** micro-frontends (MFEs), shared UI libraries, and backend microservices.

---

## 📦 Available Workflows

| Workflow | File Path | Use Case |
| :--- | :--- | :--- |
| **Deploy MFE to GCS** | [`.github/workflows/deploy-mfe-gcs.yml`](.github/workflows/deploy-mfe-gcs.yml) | Builds Vite/Webpack/Next.js MFEs, authenticates with Google Cloud, syncs bundle to GCS bucket (`gs://mycommerce/`), and applies strict `no-cache` headers on `remoteEntry.js`. |
| **MFE CI Validation** | [`.github/workflows/mfe-ci.yml`](.github/workflows/mfe-ci.yml) | Installs dependencies, runs TypeScript type-checking (`tsc --noEmit`), and verifies production builds for Pull Requests. |

---

## 🚀 Quickstart: How to Use in Any MFE Repository

### 1. Reusable MFE Deployment to Google Cloud Storage

Add this file to your MFE repository at `.github/workflows/deploy.yml`:

```yaml
name: Deploy MFE

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  deploy:
    uses: dipeshsingh2012/shared-workflows/.github/workflows/deploy-mfe-gcs.yml@main
    with:
      dest_dir: 'mfes/<your-mfe-name>'   # e.g. mfes/testimonials-ui, mfes/cart-ui, mfes/counter-check
      bucket: 'mycommerce'              # optional, defaults to 'mycommerce'
      node_version: '20'                # optional, defaults to '20'
      build_command: 'npm run build'    # optional, defaults to 'npm run build'
      dist_dir: 'dist'                  # optional, defaults to 'dist'
    secrets:
      gcp_sa_key: ${{ secrets.GCP_SA_KEY }}
```

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

## ⚙️ Workflow Inputs & Secrets

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

#### Secrets (`secrets:`)
| Name | Required | Description |
| :--- | :---: | :--- |
| `gcp_sa_key` | Optional* | JSON string of the GCP Service Account Key (`roles/storage.objectAdmin`). |
| `gcp_workload_identity_provider` | Optional* | Workload Identity Provider resource URI (if not using key). |
| `gcp_service_account` | Optional | Service account email when using Workload Identity. |

*\*At least one authentication mechanism (`gcp_sa_key` or `gcp_workload_identity_provider`) is required.*

---

## 🔒 Security & Access Configuration

For private repositories to call workflows in this repository:
1. Go to **Settings > Actions > General** in the `shared-workflows` repository.
2. Scroll to **Access**.
3. Select **Accessible from repositories in the 'dipeshsingh2012' organization / user account**.
*(If the repository is public, it works automatically without any extra permissions configuration).*
