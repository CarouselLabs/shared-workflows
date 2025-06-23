# Shared Workflows

This repository contains reusable GitHub Actions workflows for tenant deployments.

## Available Workflows

### Frontend Deployment

```yaml
jobs:
  call-frontend-workflow:
    uses: CarouselLabs/shared-workflows/.github/workflows/reusable-frontend-deploy.yml@main
    with:
      tenant: ${{ github.event.inputs.tenant }}
      stage: ${{ github.event.inputs.stage }}
      service_name: ${{ github.event.inputs.service_name }}
      website_subdomain: ${{ github.event.inputs.website_subdomain }}
      aws-account-id: ${{ github.event.inputs.aws-account-id }}
```

### Serverless Deployment

```yaml
jobs:
  call-serverless-workflow:
    uses: CarouselLabs/shared-workflows/.github/workflows/reusable-serverless-deploy.yml@main
    with:
      tenant: ${{ github.event.inputs.tenant }}
      stage: ${{ github.event.inputs.stage }}
      service_name: ${{ github.event.inputs.service_name }}
      aws_region: ${{ github.event.inputs.aws_region || 'eu-west-1' }}
    secrets:
      ROLE_TO_ASSUME: ${{ secrets.ROLE_TO_ASSUME }}
```
