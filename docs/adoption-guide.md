# Guia de Adoção

Este guia mostra como adotar o repositório central de pipelines nos repositórios
dos serviços.

## 1. Mantenha wrappers de workflow em cada serviço

Cada serviço deve manter apenas arquivos simples em `.github/workflows/*.yml`. A
implementação real fica neste repositório e é referenciada com `uses:`.

Como o `fcg-pipelines` é privado, habilite o acesso para os repositórios de
serviço antes da adoção:

- Configurações do repositório `fcg-pipelines`
- Actions
- General
- Access
- Allow repositories in the organization to use these reusable workflows

Use `examples/` como base para:

- `fcg-catalog`
- `fcg-users`
- `fcg-payments`
- `fcg-orchestration`

## 2. Configure GitHub Environments

Crie pelo menos um ambiente protegido chamado `production` em cada repositório.

Proteções recomendadas:

- Revisores obrigatórios para aprovação manual ao fazer deploy em produção.
- Restrição de branch para `main`.
- Secrets de ambiente para acesso à cloud e tokens de produto.

## 3. Use Azure OIDC em vez de client secrets

Crie uma aplicação no Azure AD com credenciais federadas para cada repositório ou
para a organização. Armazene apenas os IDs nos secrets do GitHub:

- `AZURE_CLIENT_ID`
- `AZURE_TENANT_ID`
- `AZURE_SUBSCRIPTION_ID`

Nenhum client secret do Azure é necessário quando o OIDC está configurado
corretamente.

## 4. Use Kubernetes ou cofres gerenciados em runtime

Segredos da aplicação não devem ser commitados no código-fonte nem nos arquivos
de workflow.

Fontes recomendadas em runtime:

- Kubernetes Secrets
- Azure Key Vault with workload identity
- Managed identities

Exemplos de valores sensíveis:

- Strings de conexão SQL
- Credenciais de MongoDB ou DynamoDB
- Strings de conexão Redis
- Credenciais de OpenSearch ou Elasticsearch
- Credenciais SASL do Kafka
- Chaves de assinatura JWT

## 5. Prefira GitOps para produção

Para a arquitetura final, mantenha os manifests Kubernetes no
`fcg-orchestration` e deixe Argo CD ou Flux reconciliar as mudanças no AKS.

O workflow reutilizável `gitops-image-update.yml` atualiza referências de imagem
em um kustomization. O deploy direto no AKS continua disponível para uma
demonstração simples de live deploy.

## 6. Divisão sugerida dos workflows

Pull requests:

- Política de nome de branch
- Restore
- Build
- Testes unitários
- Testes integrados
- Testes funcionais quando disponíveis
- Limite mínimo de cobertura
- Secret scan
- Análise de vulnerabilidades em dependências
- Validação de build Docker
- SonarCloud analysis

Branch `main`:

- Repetir os gates de CI
- Fazer build da imagem Docker
- Fazer scan da imagem Docker
- Enviar imagem para o ACR
- Fazer deploy no AKS ou atualizar manifests GitOps
- Executar healthcheck
