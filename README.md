# GitHub Workflows Collection

A collection of reusable GitHub Actions workflows for deploying applications to various platforms including Vercel and Google Cloud Platform (GCP).

## 📋 Overview

This repository contains production-ready GitHub Actions workflow templates for:
- **Vercel Deployments** - Deploy Express.js applications to Vercel
- **Google Cloud Run Deployments** - Multiple deployment strategies for GCP

## 🚀 Available Workflows

### 1. Express.js to Vercel (`express-to-vercel.yml`)

Deploy Express.js applications to Vercel with automatic production and preview deployments.

**Features:**
- 🔄 Automatic deployment on push to main branch
- 🔍 Preview deployments for pull requests
- 🧪 Optional test execution before deployment
- ⚡ Built with Bun for fast builds

**Usage:**

```yaml
# Copy to .github/workflows/deploy-vercel.yml
name: Deploy to Vercel

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  workflow_dispatch:

# ... rest of the workflow
```

**Required Secrets:**
- `VERCEL_TOKEN` - Your Vercel authentication token
- `VERCEL_ORG_ID` - Your Vercel organization ID
- `VERCEL_PROJECT_ID_CE_SERVER` - Your Vercel project ID

**Required Structure:**
- Application must be in `apps/server` directory
- Must have `build` script in package.json
- Optional: `test` script for running tests

---

### 2. Express.js to GCP with Docker (`express-to-gcp-with-docker.yml`)

Deploy applications to Google Cloud Run using pre-built Docker images from Google Container Registry.

**Features:**
- 🐳 Docker image building and pushing to GCR
- 📊 Intelligent change detection (only deploys when code changes)
- 🔄 Supports main and staging branches
- ⚙️ Automatic Cloud Run deployment

**Usage:**

```yaml
# Copy to .github/workflows/deploy-gcp-docker.yml
name: Deploy to GCP with Docker

on:
  push:
    branches: [main, staging]
  workflow_dispatch:

# ... rest of the workflow
```

**Required Secrets:**
- `CREDENTIALS_JSON` - GCP service account credentials (JSON)
- `GCP_PROJECT_ID` - Your GCP project ID
- `GCP_PROJECT_REGION` - GCP region for deployment (e.g., `us-central1`)

**Required Structure:**
- Application must be in `apps/server` directory
- Must have a `Dockerfile` in `apps/server/Dockerfile`

---

### 3. Express.js to GCP from Source (`express-to-gcp-with-source.yml`)

Deploy applications directly to Google Cloud Run from source code without using Docker images.

**Features:**
- 🚀 Direct source deployment (no Docker build required)
- ⚡ Faster for simple applications
- 📊 Change detection for efficient deployments
- 🎯 Manual trigger support

**Usage:**

```yaml
# Copy to .github/workflows/deploy-gcp-source.yml
name: Deploy to GCP from Source

on:
  workflow_dispatch:

# ... rest of the workflow
```

**Required Secrets:**
- `CREDENTIALS_JSON` - GCP service account credentials (JSON)
- `GCP_PROJECT_REGION` - GCP region for deployment

**Required Structure:**
- Application must be in `apps/server` directory
- Cloud Run will automatically detect the runtime

---

### 4. Express.js to GCP with Remote Container Registry (`express-to-gcp-with-remote-container-registry.yml`)

Deploy to GCP Cloud Run using GitHub Container Registry (GHCR) as a remote repository source.

**Features:**
- 📦 Build and push to GitHub Container Registry
- 🔗 Connect GHCR to GCP Artifact Registry as remote repository
- 🔐 Automatic secret management in GCP Secret Manager
- 🏷️ Docker metadata and tagging strategy
- 🌿 Branch-specific deployments

**Usage:**

```yaml
# Copy to .github/workflows/deploy-gcp-ghcr.yml
name: Deploy to GCP via GHCR

on:
  push:
    branches: [main, staging]
  workflow_dispatch:

# ... rest of the workflow
```

**Required Secrets:**
- `CREDENTIALS_JSON` - GCP service account credentials (JSON)
- `GCP_PROJECT_ID` - Your GCP project ID
- `GCP_PROJECT_NUMBER` - Your GCP project number
- `GHCR_TOKEN` - GitHub Personal Access Token with packages:write permission

**Required Variables:**
- `GCP_REPO_REGION` - GCP Artifact Registry region (e.g., `us-central1`)
- `GCP_SERVING_REGION` - GCP Cloud Run serving region

**Required Structure:**
- Application must be in `apps/server` directory
- Must have a `Dockerfile` in `apps/server/Dockerfile`

---

## 🛠️ Setup Instructions

### Prerequisites

1. **For Vercel Deployments:**
   - Vercel account
   - Vercel CLI installed locally (optional, for testing)
   - Bun runtime (or modify workflow to use Node.js)

2. **For GCP Deployments:**
   - Google Cloud Platform account
   - GCP project with Cloud Run API enabled
   - Service account with appropriate permissions
   - Billing enabled on GCP project

### Setting Up Secrets

#### GitHub Repository Secrets

Go to your repository → Settings → Secrets and variables → Actions → New repository secret

**For Vercel:**
```
VERCEL_TOKEN=<your-vercel-token>
VERCEL_ORG_ID=<your-org-id>
VERCEL_PROJECT_ID_CE_SERVER=<your-project-id>
```

**For GCP:**
```
CREDENTIALS_JSON=<service-account-json>
GCP_PROJECT_ID=<project-id>
GCP_PROJECT_REGION=<region>
GCP_PROJECT_NUMBER=<project-number>  # For GHCR workflow
```

**For GHCR:**
```
GHCR_TOKEN=<github-personal-access-token>
```

#### GitHub Repository Variables

Go to your repository → Settings → Secrets and variables → Actions → Variables

```
GCP_REPO_REGION=us-central1
GCP_SERVING_REGION=us-central1
```

### Creating GCP Service Account

```bash
# Create service account
gcloud iam service-accounts create github-actions \
    --display-name="GitHub Actions"

# Grant necessary permissions
gcloud projects add-iam-policy-binding PROJECT_ID \
    --member="serviceAccount:github-actions@PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/run.admin"

gcloud projects add-iam-policy-binding PROJECT_ID \
    --member="serviceAccount:github-actions@PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/iam.serviceAccountUser"

gcloud projects add-iam-policy-binding PROJECT_ID \
    --member="serviceAccount:github-actions@PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/storage.admin"

# Create and download key
gcloud iam service-accounts keys create key.json \
    --iam-account=github-actions@PROJECT_ID.iam.gserviceaccount.com
```

## 📁 Repository Structure

```
github-workflows/
├── deployments/
│   ├── express-to-vercel.yml
│   ├── express-to-gcp-with-docker.yml
│   ├── express-to-gcp-with-source.yml
│   └── express-to-gcp-with-remote-container-registry.yml
└── README.md
```

## 🎯 How to Use

1. **Choose a workflow** that matches your deployment needs
2. **Copy the workflow file** to your repository's `.github/workflows/` directory
3. **Configure the required secrets** in your repository settings
4. **Adjust the workflow** if needed (paths, branch names, etc.)
5. **Push to your repository** and the workflow will run automatically

### Example: Setting up Vercel Deployment

```bash
# 1. Create workflows directory
mkdir -p .github/workflows

# 2. Copy the workflow
cp deployments/express-to-vercel.yml .github/workflows/

# 3. Configure secrets in GitHub UI
# - VERCEL_TOKEN
# - VERCEL_ORG_ID
# - VERCEL_PROJECT_ID_CE_SERVER

# 4. Commit and push
git add .github/workflows/express-to-vercel.yml
git commit -m "Add Vercel deployment workflow"
git push
```

## 🔧 Customization

### Changing Working Directory

If your application is not in `apps/server`, update the paths:

```yaml
defaults:
  run:
    working-directory: path/to/your/app
```

### Adding Environment Variables

Add environment variables to your deployment:

```yaml
- name: Deploy to Vercel
  run: vercel deploy --prod --token=${{ secrets.VERCEL_TOKEN }}
  env:
    DATABASE_URL: ${{ secrets.DATABASE_URL }}
    API_KEY: ${{ secrets.API_KEY }}
```

### Conditional Deployments

Deploy only when specific files change:

```yaml
on:
  push:
    branches: [main]
    paths:
      - 'apps/server/**'
      - 'packages/**'
```

## 🤝 Contributing

Contributions are welcome! If you have additional workflow templates or improvements:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/new-workflow`)
3. Add your workflow to the `deployments/` directory
4. Update this README with documentation
5. Commit your changes (`git commit -m 'Add new workflow'`)
6. Push to the branch (`git push origin feature/new-workflow`)
7. Open a Pull Request

## 📝 License

This project is open source and available under the [MIT License](LICENSE).

## 🐛 Issues and Support

If you encounter any issues or have questions:
- Open an issue in this repository
- Check existing issues for solutions
- Provide detailed information about your setup and the problem

## 🔗 Useful Links

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Vercel Documentation](https://vercel.com/docs)
- [Google Cloud Run Documentation](https://cloud.google.com/run/docs)
- [Docker Documentation](https://docs.docker.com/)

## ⭐ Acknowledgments

These workflows are designed to be production-ready and follow best practices for CI/CD deployments.

---

Made with ❤️ for the developer community
