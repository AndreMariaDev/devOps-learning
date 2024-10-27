## Infra as Code

* Declaraçao de todos os recursos via código.
* Automatiza todo o fluxo de criação, edição e remoção de um recurso.


## GitOps
* Incorporar fluxos SCM no contexto operacional
* Controle de Versão em todo Fluxo


Overview IAC

![](IOC-Fluxo.png)

### Declarativo vs Imperativo.

Declarativo:
* Define o estado desejado
* Engloda todos os recursos do fluxo
* Matém estados passados no histórico
* Facilita possíveis declarações futuras

Imperativo:
* Define os comandos para criar o recurso
* Necessário execução em ordem
* Em alguns casos é possível manter o histórico dpo que foi feito


## AWS

## Terraform

https://developer.hashicorp.com/terraform

### Automate infrastructure on any cloud with Terraform
https://www.terraform.io/

Principais pontos Steps :
* Provedor: Usado para gerenciar. (https://registry.terraform.io/browse/providers)
* Resource: Pretence a um Provedor. (AWS, GCP, Azure ...)
* Modulo: Template de casos e necessidades (https://registry.terraform.io/browse/modules)

## C.L.I. Terraform

https://developer.hashicorp.com/terraform/cli


## AWS SSO

Instalar o AWS CLI
https://aws.amazon.com/pt/cli/


Acessar o POrtal da AWS 

![](SSO-AWS.png)

![](enable.png)

https://us-east-1.console.aws.amazon.com/iam/home?region=us-east-1#/security_credentials


![](painel-sso.png)

Configurando o SSO Profile

1. Configuração

    Após a criação do IAC SSO vamos configurar o SSO profile.

    Para isso vamos rodar o seguinte comando no terminal: `aws configure sso`

    Aqui o camando gera uma sequencia de perguntas. As respontas estão na sessão: **Resumo das configurações** do **IAM Identity Center**

    ![](resumo-das-configuracoes.png)

2. Terraform Profile

Para configurar o terraform profile vamos acessar o link : https://registry.terraform.io/browse/providers

![](terraform-profile-site.png)

Em seguida selecionar AWS:

![](terraform-profile-site-aws.png)

Em seguida selecionar o botão `USER PROVIDER`

![](terraform-profile-site-aws-how-to-use.png)
Abaixo temos o exemplo :

```hcl
terraform {
  required_providers {
    aws = {
      source = "hashicorp/aws"
      version = "5.73.0"
    }
  }
}

provider "aws" {
  profile = "VALOR-GERADO-NO-PASSO-1"
  region  = "VALOR-CONTIDO-NO-[Resumo das configurações]"
}
```
3. Criando um projeto exemplo

* Vamos criar uma pasta com nome `devops-iac`

* Abra a pasta no VS code

* Crie um arquivo providers.tf na raiz

* NO terminal vamos rodar o seguinte comando :  `terraform init`. Aqui o terraform cria alguns arquivos de inicialização baseado em informações contidas no `providers.tf`.

4. Criando nosso primeiro recurso - AWS

Para criar o primeiro recurso vamos seguir o passos:

* Crie um arquivo main.tf
* Adicione o recuro como o do exemplo:

aqui temos : 
```hcl
resource "aws_s3_bucket" "s3_bucket"{
    bucket = "rocketseat-bucket-iac-andre-maria"
}
```
resource "aws_s3_bucket" "s3_bucket":

    `aws_s3_bucket` é o tipo de recurso que está sendo criado, indicando que será um bucket no Amazon S3.
    `s3_bucket` é o nome do recurso dentro do Terraform. Este nome é usado para referenciar este recurso em outras partes do código.

bucket = "rocketseat-bucket-iac-andre-maria":

    Define o nome do bucket S3 na AWS. Neste caso, o bucket será chamado `rocketseat-bucket-iac-andre-maria`. Este nome deve ser único em toda a AWS, pois os nomes de buckets S3 são globais.

* Verificar se a sintax está correta: `terraform validate`

* Rodar o camando de pre planejamento : `terraform plan`

* Rodar e comando de execução : 

    a- necessário aprovar
        `terraform apply` (com etapa de confirmação)

    b- `terraform apply -auto-approve` (sem etapa de confirmação)


![](bucket-01.png)

Caso queira remover :

	1- terraform plan --destroy
	2- terraform apply --destroy

5. WorkSpace

O WorkSpace possui um relacionamento one-to-many
onde um state seria associado a muitos WorkSpace.

Exemplo: Ambiente WorkSpace: `[dsv, hml, prd]`

* Para saber o workspace atual
`terraform workspace show`

* Para criar um workspace
`terraform workspace new dsv`

* Para listar 
`terraform workspace list`
    * default
    * dsv

* Para selecionar o workspace
`terraform workspace select default`

6. DataSource

* Crie um arquivo `datasources.tf`
* Adicione como o do exemplo:

```hcl
data "aws_s3_bucket" "s3_bucket_data" {
    bucket = "rocketseat-bucket-iac-andre-maria-${terraform.workspace}"
}
```
* Explicação dos elementos:

    data "aws_s3_bucket" "s3_bucket_data":

    * "aws_s3_bucket" indica que estamos usando um bloco de dados para buscar informações sobre um recurso existente, no caso, um bucket do Amazon S3.
    * "s3_bucket_data" é o nome dado a essa referência de dados no Terraform. Ele será usado para acessar as informações recuperadas do bucket em outras partes do código.

    bucket = "rocketseat-bucket-iac-andre-maria-${terraform.workspace}":

    * Especifica o nome do bucket do qual queremos obter informações.
    * "rocketseat-bucket-iac-andre-maria" é uma parte fixa do nome do bucket.
    * ${terraform.workspace} insere dinamicamente o nome do workspace atual do Terraform, permitindo que o nome do bucket seja específico para cada ambiente (por exemplo, desenvolvimento, teste ou produção).

7. Output (Saidas dos comandos apply)

* Crie um arquivo `outputs.tf`
* Adicione como o do exemplo:

```hcl
output bucket_domain_name {
  value       = data.aws_s3_bucket.s3_bucket_data.bucket_domain_name
  sensitive   = false
  description = "Nome de domínio do bucket S3"
  depends_on  = []
}

output bucket_region {
  value       = data.aws_s3_bucket.s3_bucket_data.region
  sensitive   = false
  description = "Regioão do Bucket"
  depends_on  = []
}
```

* Explicação dos elementos:

    output "bucket_domain_name":

    * Define uma saída chamada bucket_domain_name. Essa saída ficará disponível após a execução do Terraform, podendo ser utilizada em outras partes do código ou acessada para fins de visualização.

    value = data.aws_s3_bucket.s3_bucket_data.bucket_domain_name:

    * Atribui o valor a ser exibido na saída. Neste caso, o valor vem de data.aws_s3_bucket.s3_bucket_data.bucket_domain_name, que representa o nome de domínio do bucket S3 previamente buscado.
    * data.aws_s3_bucket.s3_bucket_data refere-se aos dados recuperados pelo bloco de dados aws_s3_bucket chamado s3_bucket_data.
    * bucket_domain_name é um atributo do recurso S3 que retorna o domínio público completo do bucket.

    sensitive = false:

    * Indica que o valor desta saída não é sensível (como uma senha ou chave de acesso). Quando sensitive é false, o valor pode ser exibido livremente no terminal ou nos arquivos de estado do Terraform.

    description = "Nome de domínio do bucket S3":

    * Fornece uma descrição que explica o que esta saída representa, ajudando a documentar o código.

    depends_on = []:

    * Este campo opcional pode ser usado para declarar explicitamente as dependências que a saída tem em relação a outros recursos ou dados no Terraform. Neste caso, ele está vazio, o que significa que não há dependências adicionais além das inferidas automaticamente.

* Verificar se a sintax está correta: `terraform validate`

* Rodar o camando de pre planejamento : `terraform plan`

* Rodar e comando de execução : 

    a- necessário aprovar
        `terraform apply` (com etapa de confirmação)

    b- `terraform apply -auto-approve` (sem etapa de confirmação)

8. Variable

Para evitar que tanha-mos código duplicados utilizamos o recurso de veriáveis.


Temos dois tipos de váriaveis 
`Configurativas` : (constantes) ou 
`Dinâmicas` 



* Crie um arquivo `variables.tf`
* Adicione como o do exemplo:

```hcl
variable "org_name" {
  type        = string
  default     = "rocketseat"
  description = "description"
}

```

* Explicação dos elementos:

    variable "org_name":

    * Define uma variável chamada org_name. Esse nome será utilizado em outras partes do código para referenciar o valor associado a essa variável.

    type = string:

    * Especifica o tipo de dado da variável. Neste caso, o tipo é string, ou seja, a variável deve conter um valor textual.

    default = "rocketseat":

    * Define o valor padrão da variável. Se nenhum valor for fornecido durante a execução do Terraform, o valor padrão será "rocketseat".
    * Esse valor pode ser sobrescrito ao passar outro valor para a variável via linha de comando, arquivos de variáveis ou outras fontes de entrada do Terraform.

    description = "description":

    * Fornece uma descrição para a variável, que explica o seu propósito ou uso. No exemplo fornecido, a descrição é genérica ("description"), mas é uma prática recomendada usar descrições informativas para documentar o que a variável representa, como "Nome da organização".


Agora com a variavel `org_name` criada podemos altera os códigos da aplicação que contém `"rocketseat"`

Datasource : 

```hcl
data "aws_s3_bucket" "s3_bucket_data" {
    bucket = "${var.org_name}-bucket-iac-andre-maria-${terraform.workspace}"
}
```

Nosso primeiro recurso - AWS

```hcl
resource "aws_s3_bucket" "s3_bucket"{
    bucket = "${var.org_name}-bucket-iac-andre-maria-${terraform.workspace}"

    tags = {
        Name = "First Bucket"
        Iac = true
        context = "${terraform.workspace}"
    }
}
```

