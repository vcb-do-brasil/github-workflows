# GitHub Workflows - Terraform Deploy

Coleção de workflows reutilizáveis do GitHub Actions para deploy automatizado de infraestrutura AWS usando Terraform.

## 📋 Workflows Disponíveis

Este repositório contém 4 workflows reutilizáveis para diferentes cenários de deploy:

### 1. **Terraform Infrastructure Deploy** (`terraform-infra-deploy.yml`)
Workflow completo para gerenciamento de infraestrutura Terraform com CI/CD.

**Características:**
- Executa automaticamente em push/PR para branch `main`
- Validação de formato, inicialização e validação do Terraform
- Adiciona o plano do Terraform como comentário nos PRs
- Apply automático apenas em push para `main`
- Suporte a workflow manual (`workflow_dispatch`)

**Uso:**
```yaml
# Configurado para rodar automaticamente
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
```

### 2. **Terraform EC2 Deploy** (`terraform-ec2-deploy.yml`)
Workflow reutilizável para deploy de instâncias EC2.

**Características:**
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
2. Secrets configurados no repositório:
   - `AWS_ACCESS_KEY_ID`
   - `AWS_SECRET_ACCESS_KEY`
3. Variables configuradas:
   - `AWS_REGION` (opcional, padrão: us-east-1)

### Configuração de Secrets
1. Vá para Settings > Secrets and variables > Actions
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
    uses: seu-usuario/github-workflows/.github/workflows/terraform-lambda-deploy.yml@main
    with:
      environment: production
      function_name: my-function
    secrets: inherit
```

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

- O apply do Terraform só é executado na branch `main`
- Para PRs, apenas plan é executado e comentado
- Todos os workflows usam as versões mais recentes das actions (@v4/@v5)
- Suporte a workflow manual via `workflow_dispatch` (infraestrutura)

## 🤝 Contribuindo

1. Fork o projeto
2. Crie uma branch para sua feature (`git checkout -b feature/AmazingFeature`)
3. Commit suas mudanças (`git commit -m 'Add some AmazingFeature'`)
4. Push para a branch (`git push origin feature/AmazingFeature`)
5. Abra um Pull Request

## 📄 Licença

Este projeto está sob licença MIT.
