# FCG Pipelines

Reusable GitHub Actions workflows for the FIAP Cloud Games services.

This repository centralizes CI/CD automation so each service repository only keeps a
thin workflow file with service-specific variables. The goal is to make the
delivery model easier to maintain, auditable, and aligned with the Phase 4
requirements: cloud-native delivery, container images, security checks, AKS
deployments, and optional GitOps promotion.

## Workflows

| Workflow                                        | Purpose                                                                                                          |
| ----------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| `.github/workflows/branch-name-check.yml`       | Reusable branch naming policy.                                                                                   |
| `.github/workflows/dotnet-service-ci.yml`       | Build, tests, coverage, dependency scan, secret scan, SonarCloud, and Docker build validation for .NET services. |
| `.github/workflows/dotnet-service-delivery.yml` | Build, scan, push Docker image to Azure Container Registry, and deploy to AKS with rolling update.               |
| `.github/workflows/gitops-image-update.yml`     | Update image tags in a GitOps repository so Argo CD or Flux can reconcile the cluster.                           |
| `.github/workflows/terraform-azure.yml`         | Plan and optionally apply Azure infrastructure with Terraform using OIDC.                                        |

## Recommended service layout

Each service repository should keep only small workflow wrappers, for example:

```yaml
name: Catalog CI

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  ci:
    uses: group10-tc-01/fcg-pipelines/.github/workflows/dotnet-service-ci.yml@main
    with:
      service_name: catalog
      solution_path: FCG.Catalog.sln
      unit_tests_path: tests/FCG.Catalog.UnitTests/FCG.Catalog.UnitTests.csproj
      integration_tests_path: tests/FCG.Catalog.IntegratedTests/FCG.Catalog.IntegratedTests.csproj
      dockerfile_path: src/FCG.Catalog.WebApi/Dockerfile
      sonar_project_key: group10-tc-01_fcg-catalog
      sonar_organization: group10-tc-01
    secrets: inherit
```

## Delivery strategy

Two deployment models are supported:

1. Direct AKS rollout: build the image, push it to ACR, update the Kubernetes
   deployment, and wait for rollout status.
2. GitOps promotion: build and push the image in the service repository, then
   update the image tag in `fcg-orchestration`. Argo CD or Flux applies the
   change to the cluster.

For the Phase 4 pitch, direct AKS rollout is the fastest way to demonstrate a
live deploy. GitOps is the most professional long-term model because production
state is declared in the orchestration repository.

## Security baseline

The workflows avoid hardcoded credentials and expect secrets to come from
GitHub Environments, Azure federated credentials, Kubernetes Secrets, Azure Key
Vault, or another managed secret store.

See `docs/required-secrets-and-vars.md` for the required configuration.

## Adoption

Use `docs/adoption-guide.md` and the files under `examples/` as the starting
point for each service repository.
