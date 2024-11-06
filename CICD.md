# CI/CD

## Entrega de Valor

Sem CI/CD 
* Time de desenvolvimento desenvolvem 
* Time de infra disponibiliza (ação manual)
* Correções mais demoradas 
* Ciclos de feedback maiores

## Entrega Contínua de Valor

* Rápido as interações 
* Um círculo menor de feedback
* Visualidade 
* Automação de todo o fluxo


### CI (Continuous Integration)
O que é?
* Mesclagem regular de código no repositório Central 
* Refere-se ao estágio de construção 
* Necessário que o time tenha a cultura de integração

Qual Problema Resolve?

* Impossibilita a demora na integração do código 
* Permite feedbacks técnicos mais rápidos 
* Visibilidade

Como Funciona?

* Validações em tempo de Pull Request (builds e testes).
* A partir de um código integrado na branch main 
* Inicia-se todo o processo de construção 
* Instalação de dependências, builds e testes

### CD (Continuous Delivery)

O que é?

* Colocar tudo em produção e/ou homologação (disponibilizar o que foi feito no CI em um ambiente)
* Continuação da CI
* Código pode ser liberado gradualmente (exemplo liberar uma feature de forma gradual, 10% to tráfico para um nicho de clientes específico )

Qual Problema Resolve?

* Permite a entrega de valor de forma automatizada
* Garante facilidade em possíveis fluxos de Roll back
* Visibilidade

Como Funciona?

* Depois do fluxo de CI
* Inicia-se todo o processo de publicação
* Publicação homologação e/ou produção


# CI/CD - Ferramentas (Pipeline)

* Jenkins
* CircleCI
* AzureDevOps !
* Gitlab CI !
* GitActions !

## CI/CD - GitActions (GitHubActions)

O que é?

* Ferramenta do github que possibilita a implementação do CI/CD
* É um orquestrador de workflow/job
* Um workflow pode ter várias ações
* Elimina a necessidade de integração com serviços externos
* Na conta free do gifthub conseguimos utilizar essa ferramenta, tendo 2.000 minutos por mês para utilizar.

### Componebtes

* Workflow (Local onde é descrito todo o processo de automação)
* Actions (Tasks que contemplam o Workflow )
* Runners (responsável por rodar o Workflow)

## Configurando a primeira repositório

Aqui vamos utilizar o projeto que criamos na fase 2 utilizando o `DockerTutorial.md`

1. Criando repositório no GitHub 

https://github.com/AndreMariaDev/rocketseat.ci.api

2. GitHub Actions 

Quando navegamos no repositório do Git temos o menu de Abas principais neste menu temos o item relacionado ao Action.

![](image/git-actions.png)

Clicando no botão vamos para a página to actons :

![](image/git-actions-overview.png)

Aqui Vamos clicar em `set up a workflow yourself`

![](image/create-first-acton.png)

![](image/start-create-action.png)


Aqui vamos renomear o arquivo para ci.yml

Logo após vamos clicar no botão `Commit Changes`

![](image/btn-commit.png)

Click novamente em `Commit Changes`

![](image/pop-commit.png)

![](image/new-file-action.png)

Logo após devemos rodar o comando no terminal `git pull` para obter o novo arquivo criado no site do github.

![](image/git-pull-action.png)

Podemos reparar que no visual Studio code foi criado um novo diretório githrubi/ workfroes contendo o arquivo contendo um novo arquivo

![](image/new-folder-action.png)

Agora vamos incluir o seguinte trecho de código:

```hcl
name: CI

jobs:
  build:
    name: 'Build and Push'
    runs-on: ubuntu-latest
```

Aqui temos :

* `name: CI`: Define o nome do workflow, que nesse caso é "CI" (Continuous Integration). Esse nome é usado para identificar o processo de CI no GitHub.

* `jobs:` Este é um agrupamento de tarefas (jobs) que o GitHub Actions executará como parte do workflow. Cada job contém uma sequência de etapas a serem realizadas.

* `build:` Este é o nome de um job específico dentro do workflow. Cada job é uma unidade independente que o GitHub executa. Aqui, o job é nomeado "build".

* `name: 'Build and Push':` Nomeia o job "Build and Push", descrevendo brevemente o que ele faz. Esse nome aparecerá na interface do GitHub Actions.

* `runs-on: ubuntu-latest:` Define o ambiente de execução para o job. Aqui, ele usará a máquina virtual mais recente com Ubuntu fornecida pelo GitHub (ubuntu-latest), que contém ferramentas comuns de desenvolvimento e compilação.

Agora vamos implementar o `steps`.

`steps:` Este é o agrupamento das etapas (steps) que o job "build" executará. Cada etapa é uma ação específica.

O primeiro item que vamos declara no steps é `acesso ao código`. É necessário a declaração do `Checkout` que referencia a branch.

O `Checkout` é necessário porque ele baixa o código do repositório Git para a máquina virtual onde o workflow está sendo executado. Sem essa etapa, o workflow não teria acesso aos arquivos do repositório, o que impede a execução de ações que dependem do código.

Para configuramos o `Checkout` vamos consultar o site : https://github.com/marketplace?type=actions

![](image/checkout-actions.png)

Vamos clicar em `Checkout : https://github.com/marketplace/actions/checkout

![](image/fetch-only-single-file.png)
Logo após vamos copiar o trecho `- uses: actions/checkout@v4` e add no script 

```hcl
name: CI

jobs:
  build:
    name: 'Build and Push'
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
```

Assim temos : 

* `- uses: actions/checkout@v4:` Essa linha define uma etapa que utiliza a ação actions/checkout na versão v4. O actions/checkout é uma ação oficial do GitHub que faz o "checkout" do repositório, ou seja, baixa o código-fonte do repositório para a máquina virtual de execução (no caso, o Ubuntu). Isso é necessário para que as próximas etapas do workflow possam acessar o código.


Agora podemos realizar um commit na branch main.

``
`git add .`
`git commit -m "new: add first configuration for actions"`
`git push`

![](image\first-commit-actions-steps.png)


Após executar o commit vamos consultar o repositório no github. No site vamos navegar até a aba Action.

![](image\log-actions-error.png)

Vimos que ao executar a ação foi gerado um log de erro :

![](image\error-action-trigger.png)

Por que?

O erro "No event triggers defined in 'on'" ocorre porque o workflow YAML está sem uma seção on, que define os eventos que devem disparar o workflow.

A seção on é essencial para que o GitHub saiba em quais condições ele deve executar o workflow. 

Por exemplo, você pode configurar o workflow para ser disparado quando houver um push, pull_request, ou outro evento específico.

Logo vamos modificar o arquivo `.github\workflows\ci.yml` com o seguinte trecho.
 
```hcl
name: CI

on:
  push:
    branches:
      - main
      

jobs:
  build:
    name: 'Build and Push'
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4
```

Agora podemos realizar um commit na branch main.

``
`git add .`
`git commit -m "new: add configure trigger pipeline"`
`git push`


![](image\commit-trigger-actions.png)


![](image\run-ok-pipeline.png)


![](image\job-run-action.png)

![](image\first-step-job-action.png)


Agora vamos avançar em nosso script incluindo as configurações refrenetes a linguagem que nossa api foi desenvolvida.

Para isso vamos consultar novamente o site https://github.com/marketplace

Vamos pesquisar por `https://github.com/marketplace`

![](image\node-js-actions.png)

![](image\setup-node-js-environment.png)

Logo vamos modificar o arquivo `.github\workflows\ci.yml` com o seguinte trecho.
 
```hcl
name: CI

on:
  push:
    branches:
      - main


jobs:
  build:
    name: 'Build and Push'
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup node
        uses: actions/setup-node@v4
```

O que acontece na etapa "Setup node"?

* name: Setup node: Esta linha dá um nome para a etapa dentro do job. O nome "Setup node" descreve que essa etapa será responsável por configurar o Node.js no ambiente do workflow.

* uses: actions/setup-node@v4:
        uses: O comando uses indica que estamos utilizando uma ação já existente no GitHub Actions.
        actions/setup-node: Especifica que estamos utilizando a ação oficial setup-node, que é usada para configurar o Node.js no ambiente de execução. Ela instala uma versão do Node.js, configura o gerenciador de pacotes npm (ou yarn), e prepara o ambiente para rodar tarefas específicas relacionadas ao Node.js.
        @v4: Indica que estamos utilizando a versão 4 da ação setup-node.

O que a ação setup-node faz?

A ação setup-node prepara o ambiente de execução do GitHub Actions para usar o Node.js. Ela pode realizar as seguintes funções, dependendo de como é configurada:

* Instalar o Node.js: Instala a versão especificada do Node.js (por padrão, a última versão estável, mas você pode especificar versões específicas).
    Configuração do npm ou yarn: Se o Node.js estiver sendo instalado, a ação também configura o npm (ou yarn) para gerenciamento de pacotes.
    Cache de dependências: A ação pode ajudar a configurar o cache de dependências (como pacotes npm), o que melhora o desempenho ao evitar a necessidade de baixar as dependências toda vez que o workflow rodar.


Mas vamos utilizar a abordagem de `Matrix Testing` do Setup Node.js Environment.
https://github.com/marketplace/actions/setup-node-js-environment

```hcl

name: CI

on:
  push:
    branches:
      - main


jobs:
  build:
    name: 'Build and Push'
    runs-on: ubuntu-latest
    strategy:
      matrix:
        node: [ 18, 20 ]
    steps:
      - uses: actions/checkout@v4

      - name: Setup node | ${{ matrix.node }}
        uses: actions/setup-node@v4
        with:
          node-version: ${{ matrix.node }}
      - run: npm ci
      - run: npm test

```


* `strategy`:

    Define uma estratégia de matriz (matrix) para o job.
    matrix permite rodar o job em diferentes variações de configuração, neste caso:
        node: [18, 20]: O job será executado duas vezes, uma para cada versão de Node.js especificada (18 e 20).

Essa abordagem ajuda a garantir que o código funcione corretamente em ambas as versões do Node.js.

* `- name: Setup node | ${{ matrix.node }}`:

    Configura o Node.js para a versão especificada no matrix.

    `${{ matrix.node }}`: Esse valor é uma variável que representa a versão do Node.js que está sendo usada no momento (18 ou 20). 
    Isso permite que o GitHub Actions rode o job com cada versão listada na matriz.

    `uses: actions/setup-node@v4`: Configura o Node.js usando a ação oficial setup-node.

    `with: node-version: ${{ matrix.node }}`: Define a versão do Node.js a ser usada de acordo com a versão especificada na matriz.

* `- run: npm ci`:

    Executa o comando npm ci, que instala as dependências do projeto. Esse comando é preferido em CI/CD porque instala as dependências de forma rápida e confiável, usando o package-lock.json.

* `- run: npm test`:

    Executa os testes do projeto usando npm test. Este comando roda os testes definidos no package.json para garantir que o código está funcionando conforme esperado.


Agora podemos realizar um commit na branch main.

``
`git add .`
`git commit -m "new: add configure matrix strategy"`
`git push`


![](image\running-matrex-strategy.png)

