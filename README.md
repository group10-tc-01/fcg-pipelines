# FCG Pipelines

Workflows reutilizáveis do GitHub Actions para os serviços da FIAP Cloud Games.

Este repositório centraliza a automação de CI/CD para que cada serviço mantenha
apenas um workflow simples com as variáveis específicas da aplicação. O objetivo
é deixar o modelo de entrega mais fácil de manter, auditável e alinhado aos
requisitos da Fase 4: entrega cloud-native, imagens de container, verificações
de segurança, deploy em AKS e promoção opcional via GitOps.

## Workflows

| Workflow                                        | Finalidade                                                                                                                 |
| ----------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `.github/workflows/branch-name-check.yml`       | Política reutilizável para validação do nome das branches.                                                                 |
| `.github/workflows/dotnet-service-ci.yml`       | Build, testes, cobertura, análise de dependências, secret scan, SonarCloud e validação de build Docker para serviços .NET. |
| `.github/workflows/dotnet-service-delivery.yml` | Build, scan, push da imagem Docker para o Azure Container Registry e deploy no AKS com rolling update.                     |
| `.github/workflows/gitops-image-update.yml`     | Atualização de tags de imagem em um repositório GitOps para que Argo CD ou Flux faça a reconciliação do cluster.           |
| `.github/workflows/terraform-azure.yml`         | Execução de plan e, opcionalmente, apply da infraestrutura Azure com Terraform usando OIDC.                                |

## Estrutura recomendada nos serviços

Cada repositório de serviço deve manter apenas wrappers pequenos de workflow,
por exemplo:

Como este repositório é privado, o acesso do GitHub Actions precisa permitir que
os repositórios de serviço da organização `group10-tc-01` consumam os workflows
reutilizáveis daqui. Configure isso nas configurações do repositório ou da
organização antes de substituir os workflows existentes dos serviços.

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

## Estratégia de entrega

Dois modelos de deploy são suportados:

1. Rollout direto no AKS: faz build da imagem, envia para o ACR, atualiza o
   deployment no Kubernetes e aguarda o status do rollout.
2. Promoção via GitOps: faz build e push da imagem no repositório do serviço e,
   em seguida, atualiza a tag da imagem no `fcg-orchestration`. Argo CD ou Flux
   aplica a mudança no cluster.

Para o pitch da Fase 4, o rollout direto no AKS é a forma mais rápida de
demonstrar um live deploy. Para o longo prazo, GitOps é o modelo mais
profissional, porque o estado de produção fica declarado no repositório de
orquestração.

## Base de segurança

Os workflows evitam credenciais hardcoded e esperam que os segredos venham de
GitHub Environments, credenciais federadas do Azure, Kubernetes Secrets, Azure
Key Vault ou outro cofre de segredos gerenciado.

Consulte `docs/required-secrets-and-vars.md` para ver a configuração necessária.

## Adoção

Use `docs/adoption-guide.md` e os arquivos em `examples/` como ponto de partida
para cada repositório de serviço.

Para um exemplo completo e alinhado aos caminhos reais do Catalog, consulte
`examples/catalog/`.
