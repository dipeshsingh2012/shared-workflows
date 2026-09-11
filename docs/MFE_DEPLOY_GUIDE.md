# MFE Deployment Guide: Google Cloud Storage & Module Federation

This guide explains how each micro-frontend in the Hiljhil Roasters / MyCommerce architecture deploys its remote bundle to Google Cloud Storage.

---

## Architecture Overview

```
[ Developer Git Push ]
          │
          ▼
[ GitHub Actions Runner ]
          │
    (npm run build)
          │
          ▼
   dist/
   ├── assets/
   │   ├── remoteEntry.js  <── Cache-Control: no-cache, no-store, must-revalidate
   │   └── *.[hash].js     <── Cache-Control: public, max-age=31536000, immutable
   └── index.html          <── Cache-Control: no-cache, must-revalidate
          │
          ▼
[ gcloud storage rsync ]
          │
          ▼
[ GCS Bucket: gs://mycommerce/mfes/<mfe-name>/ ]
          │
          ▼ (HTTP/2 with CORS: *)
https://storage.googleapis.com/mycommerce/mfes/<mfe-name>/assets/remoteEntry.js
          │
          ▼
[ Storefront Host: mycommerce (Port 5170) ]
```

---

## Adding GCS Deployment to an Existing MFE

### Step 1: Add Workflow File
In the MFE repository root, create `.github/workflows/deploy.yml`:

```yaml
name: Deploy MFE to GCS

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  deploy:
    uses: dipeshsingh2012/shared-workflows/.github/workflows/deploy-mfe-gcs.yml@main
    with:
      dest_dir: mfes/<your-mfe-name>
      bucket: mycommerce
    secrets:
      gcp_sa_key: ${{ secrets.GCP_SA_KEY }}
```

### Step 2: Configure GitHub Repository Secrets
Under your MFE repository **Settings > Secrets and variables > Actions**:
1. Add Secret `GCP_SA_KEY` with the JSON content of your GCP deployer service account key.

### Step 3: Register in Host (`mycommerce`)
In `/home/dipes/projects/mycommerce`:
1. Add remote definition in `vite.config.ts`:
   ```ts
   remotes: {
     'my-mfe': isProd
       ? 'https://storage.googleapis.com/mycommerce/mfes/<your-mfe-name>/assets/remoteEntry.js'
       : 'http://localhost:<port>/assets/remoteEntry.js',
   }
   ```
2. Add type definition in `src/remotes.d.ts`.
3. Import and consume the exposed fragment in your host component.
