# Adoption Guide

This guide shows how to adopt the central pipeline repository in the service
repositories.

## 1. Keep workflow wrappers in each service repository

Each service should keep only thin `.github/workflows/*.yml` files. The real
implementation lives in this repository and is referenced with `uses:`.

Use `examples/` as the baseline for:

- `fcg-catalog`
- `fcg-users`
- `fcg-payments`
- `fcg-orchestration`

## 2. Configure GitHub Environments

Create at least one protected environment named `production` in each repository.

Recommended protections:

- Required reviewers for manual approval when deploying to production.
- Branch restriction to `main`.
- Environment secrets for cloud access and product tokens.

## 3. Use Azure OIDC instead of client secrets

Create an Azure AD application with federated credentials for each repository
or for the organization. Store only the IDs in GitHub secrets:

- `AZURE_CLIENT_ID`
- `AZURE_TENANT_ID`
- `AZURE_SUBSCRIPTION_ID`

No Azure client secret is needed when OIDC is configured correctly.

## 4. Use Kubernetes or managed secret stores at runtime

Application secrets must not be committed in source code or workflow files.

Recommended runtime sources:

- Kubernetes Secrets
- Azure Key Vault with workload identity
- Managed identities

Examples of sensitive values:

- SQL connection strings
- MongoDB or DynamoDB credentials
- Redis connection strings
- OpenSearch or Elasticsearch credentials
- Kafka SASL credentials

## 5. Prefer GitOps for production

For the final architecture, keep Kubernetes manifests in `fcg-orchestration`
and let Argo CD or Flux reconcile changes into AKS.

The reusable workflow `gitops-image-update.yml` updates image references in a
kustomization. Direct AKS deployment is still available for a simple live deploy
demo.

## 6. Suggested workflow split

Pull requests:

- Branch naming policy
- Restore
- Build
- Unit tests
- Integration tests
- Functional tests when available
- Coverage threshold
- Secret scan
- Dependency vulnerability scan
- Docker build validation
- SonarCloud analysis

Main branch:

- Repeat CI gates
- Build Docker image
- Scan Docker image
- Push image to ACR
- Deploy to AKS or update GitOps manifests
- Run healthcheck
