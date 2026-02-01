# GitHub Workflows - Terraform Deploy

Coleção de workflows reutilizáveis do GitHub Actions para deploy automatizado de infraestrutura AWS usando Terraform.

## ⚠️ Requisitos de Configuração

Este repositório contém apenas workflows reutilizáveis e **não executa workflows automaticamente** quando há mudanças no próprio repositório. Os workflows são chamados por outros repositórios.

### Proteção da Branch Main

Para garantir a qualidade do código, configure as seguintes regras de proteção na branch `main`:

1. **Acesse**: Settings > Branches > Add branch protection rule
2. **Configure**:
   - Branch name pattern: `main`
   - ✅ Require a pull request before merging
   - ✅ Require approvals (mínimo: 1)
   - ✅ Require status checks to pass before merging
   - ✅ Require branches to be up to date before merging
   - ✅ Do not allow bypassing the above settings

Estas configurações garantem que:
- PRs só sejam mesclados (merged) após aprovação
- A main só seja atualizada se os workflows forem executados com sucesso

## 📋 Workflows Disponíveis

Este repositório contém 4 workflows reutilizáveis para diferentes cenários de deploy:

### 1. **Terraform Infrastructure Deploy** (`terraform-infra-deploy.yml`)
Workflow completo para gerenciamento de infraestrutura Terraform com CI/CD.

**Características:**
- Workflow reutilizável que deve ser chamado por outros repositórios
- Validação de formato, inicialização e validação do Terraform
- Adiciona o plano do Terraform como comentário nos PRs
- Aceita apenas inputs necessários (aws_region, terraform_version, working_directory)

**Inputs:**
- `aws_region` (opcional): Região AWS (padrão: us-east-1)

**Secrets:**
- `AWS_ACCESS_KEY_ID` (obrigatório)
- `AWS_SECRET_ACCESS_KEY` (obrigatório)

**Exemplo de uso:**
```yaml
jobs:
  deploy:
    uses: ./.github/workflows/terraform-ec2-deploy.yml
    with:
      environment: production
      aws_region: us-east-1
      instance_type: t3.small
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
```

### 2. **Terraform EC2 Deploy** (`terraform-ec2-deploy.yml`)
Workflow reutilizável para deploy de instâncias EC2.

**Características:**
- Workflow reutilizável que deve ser chamado por outros repositórios
- Deploy de instâncias EC2 configuráveis
- Aguarda instância estar completamente pronta
- Deploy de aplicação via SSH (opcional)
- Suporte a Docker Compose

**Inputs:**
- `environment` (obrigatório): Ambiente de deploy
- `aws_region` (opcional): Região AWS (padrão: us-east-1)
- `instance_type` (opcional): Tipo da instância (padrão: t3.micro)

**Secrets:**
- `AWS_ACCESS_KEY_ID` (obrigatório)
- `AWS_SECRET_ACCESS_KEY` (obrigatório)
- `SSH_PRIVATE_KEY` (opcional): Para deploy via SSH

**Exemplo de uso:**
```yaml
jobs:
  deploy:
    uses: ./.github/workflows/terraform-ec2-deploy.yml
    with:
      environment: production
      aws_region: us-east-1
      instance_type: t3.small
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
```

### 3. **Terraform ECS Deploy** (`terraform-ecs-deploy.yml`)
Workflow reutilizável para deploy de aplicações containerizadas no ECS.

**Características:**
- Workflow reutilizável que deve ser chamado por outros repositórios
- Build e push de imagens Docker para ECR
- Deploy via Terraform
- Atualização forçada do serviço ECS
- Tag de imagem baseada no commit SHA

**Inputs:**
- `environment` (obrigatório): Ambiente de deploy
- `aws_region` (opcional): Região AWS (padrão: us-east-1)
- `cluster_name` (obrigatório): Nome do cluster ECS
- `service_name` (obrigatório): Nome do serviço ECS

**Secrets:**
- `AWS_ACCESS_KEY_ID` (obrigatório)
- `AWS_SECRET_ACCESS_KEY` (obrigatório)
- `ECR_REPOSITORY` (obrigatório): Nome do repositório ECR

**Exemplo de uso:**
```yaml
jobs:
  deploy:
    uses: ./.github/workflows/terraform-ecs-deploy.yml
    with:
      environment: production
      cluster_name: my-cluster
      service_name: my-service
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
      ECR_REPOSITORY: ${{ secrets.ECR_REPOSITORY }}
```

### 4. **Terraform Lambda Deploy** (`terraform-lambda-deploy.yml`)
Workflow reutilizável para deploy de funções AWS Lambda.

**Características:**
- Workflow reutilizável que deve ser chamado por outros repositórios
- Suporte a múltiplos runtimes (Python, etc.)
- Instalação automática de dependências
- Criação de pacote de deployment
- Smoke test após deploy

**Inputs:**
- `environment` (obrigatório): Ambiente de deploy
- `aws_region` (opcional): Região AWS (padrão: us-east-1)
- `function_name` (obrigatório): Nome da função Lambda
- `runtime` (opcional): Runtime da Lambda (padrão: python3.11)

**Secrets:**
- `AWS_ACCESS_KEY_ID` (obrigatório)
- `AWS_SECRET_ACCESS_KEY` (obrigatório)

**Exemplo de uso:**
```yaml
jobs:
  deploy:
    uses: ./.github/workflows/terraform-lambda-deploy.yml
    with:
      environment: production
      function_name: my-lambda-function
      runtime: python3.11
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

## 🚀 Como Usar

### Pré-requisitos
1. Conta AWS com permissões adequadas
2. Secrets configurados no repositório que vai usar os workflows:
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_ACCESS_KEY`
   - Outros secrets específicos de cada workflow

### Configuração de Secrets
1. No **seu repositório** (não neste), vá para Settings > Secrets and variables > Actions
2. Adicione os secrets necessários para cada workflow

### Usando os Workflows Reutilizáveis

Crie um workflow no seu repositório que chama estes workflows:

```yaml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  deploy-lambda:
    uses: vcb-do-brasil/github-workflows/.github/workflows/terraform-lambda-deploy.yml@v1.0.0
    with:
      environment: production
      function_name: my-function
      aws_region: us-east-1
    secrets:
      AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
      AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
```

> **Nota de Segurança**: Recomenda-se usar uma tag versionada (ex: `@v1.0.0`) ou um commit SHA específico ao invés de `@main` para garantir estabilidade e segurança.

## 📦 Estrutura do Projeto

```
.github/
└── workflows/
    ├── terraform-ec2-deploy.yml      # Deploy de instâncias EC2
    ├── terraform-ecs-deploy.yml      # Deploy de containers no ECS
    ├── terraform-infra-deploy.yml    # Deploy geral de infraestrutura
    └── terraform-lambda-deploy.yml   # Deploy de funções Lambda
```

## 🔧 Tecnologias Utilizadas

- **GitHub Actions**: Automação de CI/CD
- **Terraform**: Infrastructure as Code (v1.6.0+)
- **AWS**: Plataforma cloud
  - EC2: Instâncias virtuais
  - ECS: Serviços containerizados
  - Lambda: Funções serverless
  - ECR: Registro de containers
- **Docker**: Containerização

## 📝 Notas Importantes

### Sobre os Workflows
- Todos os workflows são **reutilizáveis** (`workflow_call`) e devem ser chamados por outros repositórios
- Este repositório **não executa workflows automaticamente** quando há mudanças nos arquivos
- Os workflows aceitam apenas os inputs necessários definidos no `workflow_call`
- Não há suporte para sobrescrever configurações via variables ou outros mecanismos
- Todos os workflows usam as versões mais recentes das actions (@v4/@v5)

### Sobre o Terraform
- O apply do Terraform só é executado na branch `main` do repositório que chama o workflow
- Para PRs, apenas plan é executado e comentado (quando aplicável)

### Proteção da Branch Main
Para garantir qualidade e segurança:
- Configure branch protection rules conforme indicado na seção "Requisitos de Configuração"
- PRs devem ser aprovados antes de merge
- Workflows devem passar com sucesso antes de atualizar a main

## 🤝 Contribuindo

1. Fork o projeto
2. Crie uma branch para sua feature (`git checkout -b feature/AmazingFeature`)
3. Commit suas mudanças (`git commit -m 'Add some AmazingFeature'`)
4. Push para a branch (`git push origin feature/AmazingFeature`)
5. Abra um Pull Request

## 📄 Licença

Este projeto está sob licença MIT.
