# GLL Kubernetes Deployment Action

A centralized GitHub Action for Kubernetes deployment pipeline with issue-based approval flow and GitOps best practices.

## Features

- ✅ **Issue-based Approval Flow** - Track deployments with GitHub Issues
- ✅ **JIRA Integration** - Automatic ticket extraction and linking
- ✅ **Multi-environment Support** - Dev, Test, QA, Stage, Production
- ✅ **Slack Notifications** - Rich notifications per environment
- ✅ **EKS Deployment** - Native Kubernetes deployment to AWS EKS
- ✅ **Hook System** - Extensible with before/after hooks
- ✅ **Security First** - Configuration loaded from master branch
- ✅ **Promotion Pipeline** - Automatic cross-environment promotion

## Usage

### Basic Usage

```yaml
name: Deploy
on:
  push:
    branches: [master, develop, 'rc/**']
  issue_comment:
    types: [created]
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - uses: gll-sistemas/gx-k8s-deployment-action@v1
        with:
          github-token: ${{ secrets.GITHUB_TOKEN }}
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          eks-cluster-name: ${{ secrets.EKS_CLUSTER_NAME }}
          docker-registry: ${{ secrets.DOCKER_REGISTRY }}
          docker-repository: my-app
```

### With Custom Hooks

```yaml
- uses: gll-sistemas/gx-k8s-deployment-action@v1
  with:
    github-token: ${{ secrets.GITHUB_TOKEN }}
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    eks-cluster-name: ${{ secrets.EKS_CLUSTER_NAME }}
    docker-registry: ${{ secrets.DOCKER_REGISTRY }}
    docker-repository: my-app
    before-release-hook: ./.github/actions/before-release
    before-deployment-hook: ./.github/actions/before-deployment
    after-deployment-hook: ./.github/actions/after-deployment
```

## Configuration

Create a `deploy/deployment-config.yml` file in your repository:

```yaml
skip_validation_users:
  - admin-user

versioning:
  major: 2
  auto_increment: true
  allow_accumulation: true

jira_tracking:
  base_url: "https://your-org.atlassian.net"
  ticket_patterns:
    - YOUR-?\d+

slack_defaults:
  webhook_secret: SLACK_WEBHOOK
  channel_secret: SLACK_CHANNEL

deployment_rules:
  - target_branch_pattern: ^master$
    source_branch_patterns:
      - ^rc/.*
      - ^hotfix/.*
    targets:
      - environment: prod
        requires_approval: true
        approved_by:
          - admin-user
```

## Hooks System

The action supports extensible hooks at key points in the deployment pipeline:

### Available Hooks

- **before-release** - Executed before creating GitHub release (e.g., Docker build)
- **after-release** - Executed after creating GitHub release (e.g., notifications)
- **before-deployment** - Executed before Kubernetes deployment (e.g., S3 sync)
- **after-deployment** - Executed after successful deployment (e.g., smoke tests)

### Hook Structure

Create hooks in `.github/actions/` with standard action.yml:

```yaml
# .github/actions/before-release/action.yml
name: Before Release Hook
description: Build Docker image before release

inputs:
  release:
    description: Release version
    required: true
  environment:
    description: Target environment
    required: true
  secrets-as-json:
    description: Repository secrets as JSON
    required: true

runs:
  using: composite
  steps:
    - name: Build Docker Image
      # ... your custom logic
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `github-token` | GitHub token for API access | ✅ | |
| `aws-access-key-id` | AWS access key ID | ✅ | |
| `aws-secret-access-key` | AWS secret access key | ✅ | |
| `eks-cluster-name` | EKS cluster name | ✅ | |
| `docker-registry` | Docker registry URL | ✅ | |
| `docker-repository` | Docker repository name | ✅ | |
| `deployment-config-path` | Path to deployment config | ❌ | `deploy/deployment-config.yml` |
| `aws-region` | AWS region | ❌ | `sa-east-1` |
| `before-release-hook` | Before release hook path | ❌ | `./.github/actions/before-release` |
| `before-deployment-hook` | Before deployment hook path | ❌ | `./.github/actions/before-deployment` |
| `after-deployment-hook` | After deployment hook path | ❌ | `./.github/actions/after-deployment` |
| `after-release-hook` | After release hook path | ❌ | `./.github/actions/after-release` |

## Outputs

| Output | Description |
|--------|-------------|
| `deployment-status` | Final deployment status |
| `deployment-url` | Deployed application URL |
| `release-tag` | Generated release tag |
| `issue-number` | Deployment issue number |
| `jira-tickets` | Extracted JIRA tickets |

## License

Private - GLL Sistemas