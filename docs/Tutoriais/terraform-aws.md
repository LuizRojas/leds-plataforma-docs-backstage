# Terraform na AWS — Guia Didático (Get Started – AWS)

Este guia resume e organiza os **6 tutoriais oficiais** da trilha *Get Started – AWS* do Terraform, para quem vai começar do zero: visão de IaC, instalação, criação, gerenciamento, destruição de recursos e colaboração com o HCP Terraform.

## Sumário
1. [Conceitos: o que é IaC com Terraform](#1-conceitos-o-que-é-iac-com-terraform)  
2. [Instalação do Terraform (CLI)](#2-instalação-do-terraform-cli)  
3. [Criar infraestrutura (EC2)](#3-criar-infraestrutura-ec2)  
4. [Gerenciar infraestrutura (variáveis, outputs, módulo VPC)](#4-gerenciar-infraestrutura-variáveis-outputs-módulo-vpc)  
5. [Destruir infraestrutura](#5-destruir-infraestrutura)  
6. [Colaboração com HCP Terraform (estado e execuções remotas)](#6-colaboração-com-hcp-terraform-estado-e-execuções-remotas)  
7. [Comandos essenciais](#7-comandos-essenciais)

---

## 1. Conceitos: o que é IaC com Terraform
**Infraestrutura como Código (IaC)** é gerir/provisionar infraestrutura via arquivos de configuração, versionáveis, em vez de cliques em GUI. O Terraform usa uma linguagem **declarativa** para descrever o estado desejado e aplica mudanças de forma previsível.

Conceitos-chave:  
- **Providers**: plugins que integram com APIs (AWS, Azure, GCP, Kubernetes etc).  
- **Workflow padrão**: *scope → author → init → plan → apply*.  
- **State**: arquivo de estado rastreia recursos criados.  
- **Colaboração**: backends remotos (como HCP Terraform) para compartilhar estado.

---

## 2. Instalação do Terraform (CLI)
- Instale via **Homebrew (macOS)**, **Chocolatey (Windows)** ou pacotes oficiais (Linux).  
- Verifique instalação com:
  ```bash
  terraform -version
  ```
- No Linux, use repositórios oficiais assinados.  
- Opcional: habilite autocompletar com:
  ```bash
  terraform -install-autocomplete
  ```

---

## 3. Criar infraestrutura (EC2)
### Pré-requisitos
- Terraform CLI (≥ 1.2).  
- AWS CLI configurado.  
- Conta AWS com permissões para EC2, VPC e SG na região `us-west-2`.

### Estrutura de arquivos
- `terraform.tf` – configuração do provider.  
- `main.tf` – provider + recursos.

**Exemplo resumido:**
```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.92"
    }
  }
  required_version = ">= 1.2"
}

provider "aws" {
  region = "us-west-2"
}

data "aws_ami" "ubuntu" {
  most_recent = true
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd-gp3/ubuntu-noble-24.04-amd64-server-*"]
  }
  owners = ["099720109477"]
}

resource "aws_instance" "app_server" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  tags = { Name = "learn-terraform" }
}
```

### Fluxo inicial
- `terraform fmt` — formata código.  
- `terraform validate` — valida arquivos.  
- `terraform init` — baixa providers.  
- `terraform apply` — cria a instância.  
- `terraform state list` / `terraform show` — inspeciona o estado.

---

## 4. Gerenciar infraestrutura (variáveis, outputs, módulo VPC)

### Variáveis
Arquivo `variables.tf`:
```hcl
variable "instance_name" {
  description = "Nome da instância"
  type        = string
  default     = "learn-terraform"
}

variable "instance_type" {
  description = "Tipo da instância"
  type        = string
  default     = "t2.micro"
}
```

### Outputs
Arquivo `outputs.tf`:
```hcl
output "instance_hostname" {
  value = aws_instance.app_server.private_dns
}
```

### Módulo VPC
Use módulo do **Terraform Registry** (`terraform-aws-modules/vpc/aws`) para provisionar VPC, subnets e SGs.  
Reinicialize com `terraform init`.  
Ao aplicar, a EC2 pode ser substituída (*replace*) por depender de nova VPC.

---

## 5. Destruir infraestrutura
Duas opções:  
1. Remover recursos do `.tf` e rodar `terraform apply`.  
2. Destruir tudo com:
   ```bash
   terraform destroy
   ```

---

## 6. Colaboração com HCP Terraform (estado e execuções remotas)
- Configure bloco `cloud` no `terraform.tf`:
```hcl
terraform {
  cloud {
    organization = "sua-org"
    workspaces {
      project = "Learn Terraform"
      name    = "learn-terraform-aws-get-started"
    }
  }
}
```
- Rode `terraform init` para migrar state.  
- Configure credenciais AWS no workspace remoto.  
- Execuções de `plan` e `apply` passam a rodar no HCP Terraform.

---

## 7. Comandos essenciais
- `terraform fmt` — formata código.  
- `terraform init` — inicializa diretório.  
- `terraform validate` — valida configuração.  
- `terraform plan` — mostra mudanças antes de aplicar.  
- `terraform apply` — cria/atualiza recursos.  
- `terraform destroy` — remove infraestrutura.  
- `terraform state list` / `terraform show` — inspeciona recursos geridos.

---

## Fonte
[HashiCorp Terraform – Get Started AWS](https://developer.hashicorp.com/terraform/tutorials/aws-get-started)
