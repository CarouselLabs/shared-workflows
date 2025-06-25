# Shared GitHub Actions Workflows

This repository contains reusable GitHub Actions workflows that can be used across multiple repositories in the CarouselLabs organization.

Last updated: $(date)

## Available Workflows

### 1. Reusable Frontend Deployment Workflow

**File:** `.github/workflows/reusable-frontend-deploy.yml`

This workflow handles the deployment of Next.js frontend applications to AWS using the Serverless Framework.

#### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `tenant` | The tenant to deploy for | Yes | - |
| `stage` | The stage to deploy to (dev, prod) | Yes | - |
| `service_name` | The name of the service | No | Repository name |
| `website_subdomain` | The subdomain of the website to deploy | No | Repository name |
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

jobs:
  call-reusable-workflow:
    uses: CarouselLabs/shared-workflows/.github/workflows/reusable-frontend-deploy.yml@main
    with:
      tenant: ${{ github.event.inputs.tenant || 'default' }}
      stage: ${{ github.event.inputs.stage || 'dev' }}
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
| `service_name` | The name of the service | No | Repository name |
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

jobs:
  call-reusable-workflow:
    uses: CarouselLabs/shared-workflows/.github/workflows/reusable-serverless-deploy.yml@main
    with:
      tenant: ${{ github.event.inputs.tenant || 'default' }}
      stage: ${{ github.event.inputs.stage || 'dev' }}
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

## Setting Up Service Repositories

Service repositories should create their own workflow files that reference these shared workflows. This pull-based model ensures better security and separation of concerns.

To set up a new service repository:

1. Create a `.github/workflows` directory in your repository
2. Create a `deploy.yml` file with the minimal template shown in the examples above
3. Ensure your repository has the necessary secrets for deployment

For more detailed instructions, see the [Service Workflow Template Guide](../docs/guides/service_workflow_template.md). 