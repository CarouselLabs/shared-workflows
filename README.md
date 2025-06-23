# Shared GitHub Actions Workflows

This repository contains reusable GitHub Actions workflows that can be used across multiple repositories in the CarouselLabs organization.

## Available Workflows

### 1. Reusable Frontend Deployment Workflow

**File:** `.github/workflows/reusable-frontend-deploy.yml`

This workflow handles the deployment of Next.js frontend applications to AWS using the Serverless Framework.

#### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `tenant` | The tenant to deploy for | Yes | - |
| `stage` | The stage to deploy to (dev, prod) | Yes | - |
| `service_name` | The name of the service | Yes | - |
| `website_subdomain` | The subdomain of the website to deploy | Yes | - |
| `aws-account-id` | The AWS Account ID for deployment | Yes | - |

#### Usage Example

```yaml
name: Deploy Service

on:
  push:
    branches: [ main, develop ]
  workflow_dispatch:
    inputs:
      tenant:
        description: 'The tenant ID to deploy for'
        required: true
        type: string
      stage:
        description: 'The deployment stage'
        required: true
        type: string
      website_subdomain:
        description: 'The subdomain of the website to deploy'
        required: true
        type: string
      service_name:
        description: 'The name of the service'
        required: true
        type: string
      aws-account-id:
        description: "The AWS Account ID for deployment"
        required: false
        default: "123456789012"
        type: string

jobs:
  call-reusable-workflow:
    uses: CarouselLabs/shared-workflows/.github/workflows/reusable-frontend-deploy.yml@main
    with:
      tenant: ${{ github.event.inputs.tenant }}
      stage: ${{ github.event.inputs.stage }}
      service_name: ${{ github.event.inputs.service_name }}
      website_subdomain: ${{ github.event.inputs.website_subdomain }}
      aws-account-id: ${{ github.event.inputs.aws-account-id || secrets.AWS_ACCOUNT_ID }}
    secrets: inherit
```

### 2. Reusable Serverless Deployment Workflow

**File:** `.github/workflows/reusable-serverless-deploy.yml`

This workflow handles the deployment of serverless backend services to AWS using the Serverless Framework.

#### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `tenant` | The tenant to deploy for | Yes | - |
| `stage` | The stage to deploy to (dev, prod) | Yes | - |
| `service_name` | The name of the service | Yes | - |
| `aws-account-id` | The AWS Account ID for deployment | Yes | - |

#### Usage Example

```yaml
name: Deploy Serverless Service

on:
  push:
    branches: [ main, develop ]
  workflow_dispatch:
    inputs:
      tenant:
        description: 'The tenant ID to deploy for'
        required: true
        type: string
      stage:
        description: 'The deployment stage'
        required: true
        type: string
      service_name:
        description: 'The name of the service'
        required: true
        type: string
      aws-account-id:
        description: "The AWS Account ID for deployment"
        required: false
        default: "123456789012"
        type: string

jobs:
  call-reusable-workflow:
    uses: CarouselLabs/shared-workflows/.github/workflows/reusable-serverless-deploy.yml@main
    with:
      tenant: ${{ github.event.inputs.tenant }}
      stage: ${{ github.event.inputs.stage }}
      service_name: ${{ github.event.inputs.service_name }}
      aws-account-id: ${{ github.event.inputs.aws-account-id || secrets.AWS_ACCOUNT_ID }}
    secrets: inherit
```

## OIDC Authentication with AWS

These workflows use OIDC authentication with AWS. The IAM role `github-actions-oidc-role-tenant-terraform` must be configured in AWS with the appropriate permissions and trust policy.

### Trust Policy

The trust policy for the IAM role should allow GitHub Actions to assume the role from the CarouselLabs organization:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::123456789012:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringLike": {
          "token.actions.githubusercontent.com:sub": "repo:CarouselLabs/*:*",
          "token.actions.githubusercontent.com:job_workflow_ref": [
            "CarouselLabs/shared-workflows/.github/workflows/reusable-frontend-deploy.yml@*",
            "CarouselLabs/shared-workflows/.github/workflows/reusable-serverless-deploy.yml@*"
          ]
        }
      }
    }
  ]
}
```

## Updating Service Repositories

To update all service repositories to use these shared workflows, run the `update_workflow_files.py` script:

```bash
python scripts/update_workflow_files.py
``` 