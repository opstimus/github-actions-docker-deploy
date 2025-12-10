# GitHub Actions Docker Deploy

A reusable GitHub Actions workflow for deploying Docker images from Amazon ECR using Docker Compose.

## Overview

This workflow provides a standardized way to deploy Docker containers from AWS ECR to your infrastructure using Docker Compose. It handles AWS authentication, ECR login, and deployment in a single reusable workflow.

## Prerequisites

1. **AWS ECR Repository**: Your Docker image must be stored in an Amazon ECR repository
2. **IAM Role**: An IAM role with permissions to:
   - Assume role for authentication
   - Pull images from ECR
3. **Docker Compose File**: Your repository must contain a `docker-compose.yml` file
4. **GitHub Runner**: A self-hosted or GitHub-hosted runner with Docker and Docker Compose installed

## Workflow Inputs

| Input | Required | Type | Description |
|-------|----------|------|-------------|
| `ecr_repository` | Yes | string | The ECR repository URI (e.g., `123456789012.dkr.ecr.ap-southeast-1.amazonaws.com/my-app`) |
| `image_tag` | Yes | string | The Docker image tag to deploy (e.g., `main-1a2b3c4`, `v1.0.1-1a2b3c4`) |
| `aws_region` | Yes | string | AWS region where the ECR repository is located (e.g., `ap-southeast-1`) |
| `aws_ecr_iam_role_arn` | Yes | string | ARN of the IAM role to assume for ECR access |
| `runner_labels` | Yes | string | JSON array of runner labels (e.g., `["self-hosted", "my-ec2"]`) |

## Usage

### Using in a Different Repository

If you want to use this workflow from a different repository, reference it as a job in your workflow:

```yaml
jobs:
  build-image:
    # ... your build job that outputs image_tag ...

  deploy-image:
    needs: build-image
    uses: opstimus/github-actions-docker-deploy/.github/workflows/docker-compose-ecr.yml@main
    with:
      ecr_repository: 123456789012.dkr.ecr.us-east-1.amazonaws.com/my-app
      image_tag: ${{ needs.build-image.outputs.image_tag }}
      aws_region: ap-southeast-1
      aws_ecr_iam_role_arn: arn:aws:iam::12312312:role/github-actions-role
      runner_labels: '["self-hosted", "local"]'
```

## Docker Compose Configuration

The workflow sets the `DOCKER_IMAGE` environment variable with the full ECR image reference. Your `docker-compose.yml` file should reference this variable:

```yaml
services:
  app:
    image: ${DOCKER_IMAGE}
    # ... other configuration ...
```

The workflow will automatically substitute `${DOCKER_IMAGE}` with the value: `{ecr_repository}:{image_tag}`

## Workflow Steps

1. **Configure AWS credentials**: Sets up AWS authentication using the provided IAM role
2. **Login to Amazon ECR**: Authenticates with ECR to pull private images
3. **Checkout repo**: Checks out the repository containing your `docker-compose.yml`
4. **Deploy using compose**: Runs `docker compose up -d` with the `DOCKER_IMAGE` environment variable set
