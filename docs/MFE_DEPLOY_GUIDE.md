# MFE Deployment Guide: Google Cloud Storage & Keyless Module Federation

This guide explains how each micro-frontend in the Hiljhil Roasters / MyCommerce architecture deploys its remote bundle to Google Cloud Storage keylessly without needing any secrets in child repositories.

---

## Architecture Overview

```
[ Developer Git Push (Any MFE Repo) ]
                   │
                   ▼
       [ GitHub Actions Runner ]
                   │
                   ├─► (npm install & npm run build)
                   │
                   ├─► Mint GitHub OIDC Token (id-token: write)
                   │
                   ▼
   [ GCP Security Token Service (STS) ]
                   │
 (Validates repository_owner == "dipeshsingh2012")
                   │
                   ▼
  [ Impersonates gh-actions-deployer SA ]
  (Zero secrets stored in the MFE repo!)
                   │
                   ▼
 [ gcloud storage rsync dist/ to gs://mycommerce/mfes/<mfe>/ ]
                   │
                   ▼
 https://storage.googleapis.com/mycommerce/mfes/<mfe>/assets/remoteEntry.js
                   │
                   ▼
      [ Storefront Host: mycommerce ]
```

---

## Adding GCS Deployment to Any MFE

In your MFE repository root, create `.github/workflows/deploy.yml`:

```yaml
name: Deploy MFE to GCS

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  id-token: write

jobs:
  deploy:
    uses: dipeshsingh2012/shared-workflows/.github/workflows/deploy-mfe-gcs.yml@main
    with:
      dest_dir: 'mfes/<your-mfe-name>'
```

**That is all!** No secrets need to be added to your repository settings.
