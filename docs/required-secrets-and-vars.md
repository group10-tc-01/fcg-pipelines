# Required Secrets And Variables

Use GitHub Environments to scope deployment credentials. Avoid repository-wide
secrets for production whenever possible.

## Organization or repository secrets

| Name                    | Required by                                          | Purpose                                           |
| ----------------------- | ---------------------------------------------------- | ------------------------------------------------- |
| `SONAR_TOKEN`           | `dotnet-service-ci.yml`                              | SonarCloud analysis token.                        |
| `AZURE_CLIENT_ID`       | `dotnet-service-delivery.yml`, `terraform-azure.yml` | Azure AD application client ID for OIDC.          |
| `AZURE_TENANT_ID`       | `dotnet-service-delivery.yml`, `terraform-azure.yml` | Azure tenant ID.                                  |
| `AZURE_SUBSCRIPTION_ID` | `dotnet-service-delivery.yml`, `terraform-azure.yml` | Azure subscription ID.                            |
| `GITOPS_TOKEN`          | `gitops-image-update.yml`                            | Token with write access to the GitOps repository. |

## Suggested GitHub variables

| Name                            | Example         | Purpose                        |
| ------------------------------- | --------------- | ------------------------------ |
| `AZURE_CONTAINER_REGISTRY_NAME` | `fcgregistry`   | ACR name without `azurecr.io`. |
| `AKS_RESOURCE_GROUP`            | `rg-fcg-prod`   | AKS resource group.            |
| `AKS_CLUSTER_NAME`              | `aks-fcg-prod`  | AKS cluster name.              |
| `K8S_NAMESPACE`                 | `fcg`           | Kubernetes namespace.          |
| `SONAR_ORGANIZATION`            | `group10-tc-01` | SonarCloud organization.       |

## Runtime application secrets

Do not store these values in workflow files or appsettings files committed to
source control:

- Database connection strings
- MongoDB or DynamoDB credentials
- Redis connection strings
- OpenSearch or Elasticsearch credentials
- Kafka SASL username and password
- JWT signing keys

Use Kubernetes Secrets or Azure Key Vault integration to inject them at runtime.
