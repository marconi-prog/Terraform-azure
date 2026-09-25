# Terraform + Azure

## Sobre

Este repositório é um guia prático sobre como utilizar **Terraform** para provisionar e gerenciar infraestrutura na **Microsoft Azure** através de código.

A ideia principal é entender como Terraform e Azure trabalham juntos e como podemos substituir configurações manuais feitas pelo Azure Portal por **Infrastructure as Code (IaC)**.

---

## O que é Terraform?

O **Terraform** é uma ferramenta de Infrastructure as Code (IaC) desenvolvida pela HashiCorp.

Em vez de criar recursos manualmente através de interfaces gráficas, podemos descrever nossa infraestrutura em arquivos de configuração.

Por exemplo, em vez de criar uma máquina virtual manualmente no Azure Portal, podemos declarar:

```hcl
resource "azurerm_resource_group" "example" {
  name     = "rg-example"
  location = "Brazil South"
}
```

O Terraform interpreta essa configuração e utiliza as APIs do Azure para criar o recurso.

---

## O que é Azure?

O **Microsoft Azure** é uma plataforma de computação em nuvem que fornece diversos serviços de infraestrutura e aplicação.

Entre eles:

* Virtual Machines
* Azure Kubernetes Service (AKS)
* Azure App Service
* Azure SQL Database
* Storage Account
* Virtual Network
* Load Balancer
* Azure Container Registry
* Key Vault
* Azure Functions
* Azure Monitor

O Azure fornece os recursos.

O Terraform pode ser utilizado para **provisionar e configurar esses recursos através de código**.

---

# Como Terraform e Azure trabalham juntos?

A comunicação acontece através do **Azure Provider do Terraform**.

```text
Terraform
    │
    ▼
Azure Provider
    │
    ▼
Azure API
    │
    ├── Resource Group
    ├── Virtual Network
    ├── VM
    ├── Storage
    ├── Database
    └── outros serviços
```

O Terraform não substitui o Azure.

Ele funciona como uma camada de automação que permite definir e gerenciar a infraestrutura do Azure através de código.

---

# Infrastructure as Code

Infrastructure as Code significa tratar infraestrutura como código-fonte.

Em vez de:

```text
Entrar no Azure Portal
        ↓
Criar Resource Group
        ↓
Criar rede
        ↓
Criar VM
        ↓
Configurar regras
        ↓
Configurar banco
```

Podemos ter:

```text
Código Terraform
       ↓
terraform plan
       ↓
terraform apply
       ↓
Azure
```

Isso traz algumas vantagens:

* Infraestrutura versionada com Git
* Reprodutibilidade
* Automação
* Menos configuração manual
* Facilidade para criar ambientes
* Histórico das alterações
* Integração com CI/CD

---

# Instalação

Depois de instalar o Terraform, podemos verificar a instalação:

```bash
terraform version
```

Também precisamos ter o Azure CLI:

```bash
az version
```

Login na Azure:

```bash
az login
```

Para verificar a conta atualmente selecionada:

```bash
az account show
```

Para listar as subscriptions:

```bash
az account list
```

Caso seja necessário selecionar uma subscription:

```bash
az account set --subscription "NOME-OU-ID-DA-SUBSCRIPTION"
```

---

# Estrutura básica de um projeto Terraform

Uma estrutura simples pode ser:

```text
terraform-azure/
├── main.tf
├── variables.tf
├── outputs.tf
├── providers.tf
├── terraform.tfvars
├── .gitignore
└── README.md
```

### `providers.tf`

Define o provider utilizado pelo Terraform.

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
    }
  }

  required_version = ">= 1.6.0"
}

provider "azurerm" {
  features {}
}
```

---

# Criando um Resource Group

Um Resource Group é um agrupamento lógico de recursos dentro do Azure.

Exemplo:

```hcl
resource "azurerm_resource_group" "main" {
  name     = "rg-terraform-example"
  location = "Brazil South"
}
```

Nesse exemplo, estamos declarando que queremos um Resource Group chamado:

```text
rg-terraform-example
```

na região:

```text
Brazil South
```

---

# Inicializando o Terraform

Depois de criar os arquivos:

```bash
terraform init
```

O comando inicializa o projeto e instala os providers necessários.

---

# Validando a configuração

Podemos verificar se os arquivos Terraform estão corretos:

```bash
terraform validate
```

Se estiver tudo correto, o Terraform informa que a configuração é válida.

---

# Formatando o código

Para manter os arquivos Terraform padronizados:

```bash
terraform fmt
```

---

# Terraform Plan

Antes de alterar a infraestrutura, podemos visualizar o que será criado, alterado ou removido:

```bash
terraform plan
```

Por exemplo:

```text
Plan: 1 to add, 0 to change, 0 to destroy.
```

Isso permite revisar as alterações antes de aplicá-las.

---

# Terraform Apply

Para aplicar a infraestrutura:

```bash
terraform apply
```

O Terraform solicitará confirmação antes de realizar as alterações.

Também podemos utilizar:

```bash
terraform apply -auto-approve
```

Isso aplica as alterações automaticamente.

---

# Terraform State

Um dos conceitos mais importantes do Terraform é o **State**.

O Terraform mantém informações sobre os recursos que ele gerencia.

Normalmente isso fica no arquivo:

```text
terraform.tfstate
```

O State permite que o Terraform saiba o estado atual da infraestrutura e consiga identificar diferenças entre:

```text
Configuração desejada
        +
Estado atual
        ↓
Terraform
        ↓
Alterações necessárias
```

### Importante

O arquivo `terraform.tfstate` pode conter informações sensíveis.

Por isso, em projetos reais, é comum utilizar um **Remote Backend**, como Azure Storage, para armazenar o state de forma centralizada e permitir trabalho em equipe.

---

# Terraform Destroy

Para remover os recursos criados pelo Terraform:

```bash
terraform destroy
```

O Terraform mostrará quais recursos serão destruídos antes da confirmação.

> ⚠️ Nunca execute `terraform destroy` em um ambiente de produção sem verificar cuidadosamente o plano.

---

# Variáveis

Também podemos evitar deixar valores diretamente no código.

### `variables.tf`

```hcl
variable "location" {
  description = "Azure region"
  type        = string
  default     = "Brazil South"
}
```

E utilizar:

```hcl
resource "azurerm_resource_group" "main" {
  name     = "rg-terraform-example"
  location = var.location
}
```

---

# Outputs

Podemos retornar informações importantes após o deploy.

### `outputs.tf`

```hcl
output "resource_group_name" {
  value = azurerm_resource_group.main.name
}
```

Depois:

```bash
terraform apply
```

O Terraform poderá mostrar:

```text
resource_group_name = "rg-terraform-example"
```

---

# Exemplo completo

## `providers.tf`

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
    }
  }

  required_version = ">= 1.6.0"
}

provider "azurerm" {
  features {}
}
```

## `main.tf`

```hcl
resource "azurerm_resource_group" "main" {
  name     = "rg-terraform-example"
  location = var.location
}
```

## `variables.tf`

```hcl
variable "location" {
  description = "Azure region"
  type        = string
  default     = "Brazil South"
}
```

## `outputs.tf`

```hcl
output "resource_group_name" {
  value = azurerm_resource_group.main.name
}
```

---

# Fluxo completo

O fluxo básico de trabalho fica:

```text
1. Escrever Terraform
        ↓
2. terraform init
        ↓
3. terraform fmt
        ↓
4. terraform validate
        ↓
5. terraform plan
        ↓
6. Revisar alterações
        ↓
7. terraform apply
        ↓
8. Azure cria/configura os recursos
```

---

# Terraform + Git

Uma das grandes vantagens é poder versionar a infraestrutura junto com o projeto.

Exemplo:

```text
GitHub
│
├── application/
│   └── código da aplicação
│
├── terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── providers.tf
│
└── README.md
```

Dessa forma, conseguimos acompanhar alterações na infraestrutura através do Git.

Por exemplo:

```bash
git add .
git commit -m "feat: add Azure infrastructure"
git push
```

---

# Terraform + CI/CD

Terraform também pode fazer parte de um pipeline de CI/CD.

Um fluxo possível:

```text
Developer
    ↓
Git Push
    ↓
GitHub
    ↓
CI/CD
    ↓
terraform fmt
    ↓
terraform validate
    ↓
terraform plan
    ↓
Review
    ↓
terraform apply
    ↓
Azure
```

Isso permite automatizar o provisionamento da infraestrutura.

---

# Terraform não substitui conhecimento de Azure

Aprender Terraform não significa que precisamos deixar de estudar Azure.

Terraform é a ferramenta de automação.

Azure é a plataforma onde a infraestrutura será executada.

Por exemplo:

| Azure           | Terraform                       |
| --------------- | ------------------------------- |
| Resource Group  | `azurerm_resource_group`        |
| Virtual Network | `azurerm_virtual_network`       |
| Virtual Machine | `azurerm_linux_virtual_machine` |
| Storage Account | `azurerm_storage_account`       |
| Azure SQL       | `azurerm_mssql_server`          |
| App Service     | `azurerm_linux_web_app`         |
| Key Vault       | `azurerm_key_vault`             |

Por isso, o ideal é estudar os dois conceitos em conjunto.

---

# O que aprender depois

Depois dos conceitos básicos, alguns tópicos importantes são:

* Terraform Variables
* Terraform Outputs
* Terraform State
* Remote Backend
* Terraform Modules
* Terraform Workspaces
* Azure Resource Groups
* Azure Virtual Network
* Azure Storage
* Azure Virtual Machines
* Azure App Service
* Azure SQL
* Azure Key Vault
* Azure Container Registry
* Azure Kubernetes Service
* CI/CD com Terraform
* Secrets Management
* Infrastructure as Code
* Terraform em ambientes de desenvolvimento, homologação e produção

---

# Resumo

Terraform permite transformar a infraestrutura do Azure em código.

Em vez de configurar tudo manualmente no Azure Portal, podemos declarar:

```text
O que queremos
      ↓
Terraform
      ↓
Azure Provider
      ↓
Azure API
      ↓
Infraestrutura
```

Isso torna a infraestrutura **automatizável, versionável, reproduzível e integrada ao processo de DevOps**.

O principal conceito para guardar é:

> **Azure fornece a infraestrutura. Terraform permite descrevê-la e gerenciá-la como código.**

---

## Tecnologias

* Terraform
* Microsoft Azure
* Azure CLI
* Git
* GitHub
* Infrastructure as Code
* DevOps
* CI/CD
