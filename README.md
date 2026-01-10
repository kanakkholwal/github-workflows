# GitHub Workflows Collection

An open-source collection of reusable GitHub Actions workflow templates for deploying applications to various cloud platforms and services.

## 📋 Overview

This repository provides production-ready GitHub Actions workflow templates that you can copy and customize for your projects. Whether you're deploying to Vercel, Google Cloud Platform, or other services, these templates offer battle-tested CI/CD patterns.

**Available Templates:**
- **Vercel Deployments** - Deploy web applications to Vercel with preview and production environments
- **Google Cloud Run Deployments** - Multiple deployment strategies for containerized applications on GCP

## 🚀 Available Workflows

### 1. Deploy to Vercel (`express-to-vercel.yml`)

Deploy web applications to Vercel with automatic production and preview deployments. Works with any framework supported by Vercel (Next.js, Express, React, Vue, etc.).

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

**Project Structure:**
This template assumes your application is in `apps/server` directory. Customize the `working-directory` in the workflow to match your project structure.

**Customization:**
```yaml
defaults:
  run:
    working-directory: your-app-directory  # Change this to match your structure
```

---

### 2. Deploy to GCP with Docker (`express-to-gcp-with-docker.yml`)

Deploy containerized applications to Google Cloud Run using Docker images stored in Google Container Registry. Suitable for any application that can be containerized.

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

**Project Structure:**
This template assumes your application is in `apps/server` directory with a Dockerfile. Adjust paths in the workflow to match your structure.

**Customization:**
```yaml
# Update the paths in the workflow
paths:
  - 'your-app/**'  # Trigger only on changes to your app directory

# Update Docker build context
docker build -f "your-app/Dockerfile" -t "${IMAGE_NAME}" "your-app"
```

---

### 3. Deploy to GCP from Source (`express-to-gcp-with-source.yml`)

Deploy applications directly to Google Cloud Run from source code without building Docker images locally. Cloud Run will handle the containerization automatically.

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

**Project Structure:**
This template assumes your application is in `apps/server` directory. Update the `--source` path in the workflow to match your application directory.

**Customization:**
```yaml
# Update source path
gcloud run deploy ${{ env.SERVICE_NAME_SERVER }} \
  --source ./your-app-directory \
  --region ${{ secrets.GCP_PROJECT_REGION }}
```

---

### 4. Deploy to GCP via GHCR (`express-to-gcp-with-remote-container-registry.yml`)

Deploy to GCP Cloud Run using GitHub Container Registry (GHCR) as an intermediate registry, leveraging GCP's remote repository feature for seamless integration.

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

**Project Structure:**
This template assumes your application is in `apps/server` directory with a Dockerfile. Customize the context and file paths in the workflow.

**Customization:**
```yaml
# Update Docker build context
with:
  context: ./your-app-directory
  file: ./your-app-directory/Dockerfile
```

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
├── deployments/           # Workflow template files
│   ├── express-to-vercel.yml
│   ├── express-to-gcp-with-docker.yml
│   ├── express-to-gcp-with-source.yml
│   └── express-to-gcp-with-remote-container-registry.yml
└── README.md             # This file
```

## 🎯 How to Use These Templates

### Quick Start

1. **Browse** the available workflow templates in the `deployments/` directory
2. **Choose** a workflow that matches your deployment target and strategy
3. **Copy** the workflow file to your project's `.github/workflows/` directory
4. **Customize** the workflow for your project structure and requirements
5. **Configure** the required secrets and variables in your repository settings
6. **Commit and push** - the workflow will automatically trigger based on its configuration

### Step-by-Step Example: Vercel Deployment

```bash
# 1. Navigate to your project
cd your-project

# 2. Create workflows directory if it doesn't exist
mkdir -p .github/workflows

# 3. Download or copy the workflow template
curl -o .github/workflows/deploy-vercel.yml \
  https://raw.githubusercontent.com/kanakkholwal/github-workflows/main/deployments/express-to-vercel.yml

# 4. Customize the workflow file to match your project structure
# Edit paths, working directories, and other configuration as needed

# 5. Configure secrets in GitHub
# Go to: Repository → Settings → Secrets and variables → Actions
# Add: VERCEL_TOKEN, VERCEL_ORG_ID, VERCEL_PROJECT_ID_CE_SERVER

# 6. Commit and push
git add .github/workflows/deploy-vercel.yml
git commit -m "Add Vercel deployment workflow"
git push
```

### Using as a Template Repository

You can also use this repository as a template:

1. Click "Use this template" on GitHub
2. Create your own workflows collection
3. Add your custom workflow templates
4. Share with your team or the community

## 🔧 Customization Guide

These templates are designed to be customized for your specific needs. Here are common customization patterns:

### Changing Project Structure

All templates assume an `apps/server` directory structure. Update paths to match your project:

```yaml
# Change working directory
defaults:
  run:
    working-directory: path/to/your/app

# Update path filters
on:
  push:
    paths:
      - 'your-app/**'

# Update Docker context
context: ./your-app-directory
```

### Adding Environment Variables

Inject environment variables into your deployment:

```yaml
- name: Deploy
  run: your-deploy-command
  env:
    DATABASE_URL: ${{ secrets.DATABASE_URL }}
    API_KEY: ${{ secrets.API_KEY }}
    NODE_ENV: production
```

### Conditional Deployments

Trigger deployments only when specific files or directories change:

```yaml
on:
  push:
    branches: [main]
    paths:
      - 'src/**'
      - 'package.json'
      - 'Dockerfile'
```

### Multi-Environment Setup

Deploy to different environments based on branches:

```yaml
- name: Set Environment
  run: |
    if [ "${{ github.ref }}" == "refs/heads/main" ]; then
      echo "ENV=production" >> $GITHUB_ENV
    elif [ "${{ github.ref }}" == "refs/heads/staging" ]; then
      echo "ENV=staging" >> $GITHUB_ENV
    fi

- name: Deploy
  run: deploy-to-${{ env.ENV }}
```

## 🤝 Contributing

We welcome contributions from the community! Help us expand this collection of workflow templates.

### Ways to Contribute

- **Add new workflow templates** for different platforms (AWS, Azure, Netlify, etc.)
- **Improve existing workflows** with better practices or new features
- **Fix bugs** or issues in the templates
- **Enhance documentation** with more examples and use cases
- **Share your use cases** and success stories

### Contribution Guidelines

1. **Fork** this repository
2. **Create** a feature branch (`git checkout -b feature/add-aws-workflow`)
3. **Add** your workflow template to the `deployments/` directory
4. **Document** your workflow:
   - Add a section in this README explaining the workflow
   - Include required secrets, variables, and structure
   - Provide usage examples and customization tips
5. **Test** your workflow in a real project to ensure it works
6. **Commit** your changes with clear messages
7. **Push** to your branch (`git push origin feature/add-aws-workflow`)
8. **Open** a Pull Request with a description of your changes

### Template Naming Convention

Use descriptive names that indicate the source type and destination:
- `[framework/app-type]-to-[platform]-[optional-method].yml`
- Examples: `nextjs-to-vercel.yml`, `django-to-aws-ecs.yml`, `react-to-netlify.yml`

## 📝 License

This project is open source and available under the [MIT License](LICENSE).

Feel free to use these workflows in your personal or commercial projects. Attribution is appreciated but not required.

## 🐛 Issues and Support

### Reporting Issues

If you encounter problems with any workflow template:

1. **Check existing issues** to see if it's already reported
2. **Provide details**:
   - Which workflow template you're using
   - Your project structure and framework
   - Error messages or unexpected behavior
   - Steps to reproduce the issue
3. **Open a new issue** with a descriptive title

### Getting Help

- **Discussions**: Use GitHub Discussions for questions and general help
- **Issues**: For bug reports and specific problems with workflows
- **Pull Requests**: For contributing fixes or improvements

## 🔗 Useful Links

- [GitHub Actions Documentation](https://docs.github.com/en/actions)
- [Vercel Documentation](https://vercel.com/docs)
- [Google Cloud Run Documentation](https://cloud.google.com/run/docs)
- [Docker Documentation](https://docs.docker.com/)

## ⭐ Why This Project?

This repository was created to provide developers with:
- **Ready-to-use** workflow templates that actually work in production
- **Best practices** for CI/CD across different platforms
- **Time savings** by not having to write workflows from scratch
- **Learning resources** for GitHub Actions patterns

## 🎓 Learning Resources

New to GitHub Actions? Check out these resources:
- [GitHub Actions Quickstart](https://docs.github.com/en/actions/quickstart)
- [Workflow Syntax Reference](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions)
- [GitHub Actions Best Practices](https://docs.github.com/en/actions/security-guides/security-hardening-for-github-actions)

## 📊 Project Status

This is an active open-source project. We regularly:
- Add new workflow templates
- Update existing workflows for new platform features
- Review and merge community contributions
- Maintain compatibility with the latest GitHub Actions features

**Star this repository** to stay updated with new workflows and improvements!

---

Made with ❤️ by the community, for the community
