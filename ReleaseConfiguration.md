# Configurando releases

## 1 Configurando estrutura

Relese é o controle de versão da sua aplicação.

```hcl
{
  "name": "rocketseat.ci.api",
  "version": "0.0.1",
  "lockfileVersion": 3,
  "requires": true,

  ....
```
O campo `"version": "0.0.1"` em um projeto de software, especialmente em apps e bibliotecas, segue a convenção de **versionamento semântico** (ou *Semantic Versioning*). Essa convenção ajuda a comunicar de forma clara as mudanças e atualizações de uma aplicação. A versão é composta por três números separados por pontos:

- **MAJOR** (o primeiro número): Indica mudanças significativas, como alterações que podem quebrar a compatibilidade com versões anteriores. Exemplo: de `1.0.0` para `2.0.0`.

- **MINOR** (o segundo número): Indica a adição de funcionalidades novas que não quebram a compatibilidade com versões anteriores. Exemplo: de `1.0.0` para `1.1.0`.

- **PATCH** (o terceiro número): Indica correções de bugs e pequenas melhorias que não afetam a compatibilidade com versões anteriores. Exemplo: de `1.0.0` para `1.0.1`.

Portanto, a versão `"0.0.1"` normalmente significa:

- **0 (MAJOR)**: Indica que a aplicação ainda está em uma fase inicial de desenvolvimento e pode ter mudanças significativas antes de se considerar estável.
- **0 (MINOR)**: Indica que ainda não foram adicionadas funcionalidades consideráveis.
- **1 (PATCH)**: Indica que houve uma pequena correção ou ajuste.

Em resumo, `"0.0.1"` é tipicamente usada para um estado inicial de desenvolvimento, com apenas correções mínimas aplicadas.

#### Configurando o release na pipeline

Para cuidar desse ponto em nossa aplicação e pipeline vamos incluir uma action em nosso arquivo `ci.yml`.

https://github.com/cycjimmy/semantic-release-action

![](image/ReleseConfiguration/basic-usege-release.png)

Vamos copiar o trecho de código e implementar em  nosso pipeline

```hcl


name: CI

on:
  push:
    branches:
      - main

permissions:
  id-token: write
  contents: read

jobs:
  build:
    name: 'Build and Push'
    runs-on: ubuntu-latest
    # strategy:
    #   matrix:
    #     node: [ 18, 20 ]
    steps:
      - uses: actions/checkout@v4

      # - name: Setup node | ${{ matrix.node }}
      - name: Setup node
        uses: actions/setup-node@v4
        with:
          # node-version: ${{ matrix.node }}
          node-version: 18
      - run: npm ci
      - run: npm test

      - name: Semantic Release
        uses: cycjimmy/semantic-release-action@v4
        env:
          GITHUB_TOKEN: ${{ secrets.GH_TOKEN_RELEASE }}

      - name: Generate tag
        id: generate_tag
        run: |
          SHA=$(echo $GITHUB_SHA | head -c7)
          echo "sha=$SHA" >> $GITHUB_OUTPUT
      
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-region: us-east-1
          role-to-assume: arn:aws:iam::***************:role/ecr_role

      - name: Login to Amazon ECR
        id: login-ecr
        uses: aws-actions/amazon-ecr-login@v2

      - name: Build docker image
        id: build-docker-image
        env:
          REGISTRY: ${{ steps.login-ecr.outputs.registry }}
          TAG: ${{ steps.generate_tag.outputs.sha }}
        run: |
          docker build -t $REGISTRY/rocketseat-ci:$TAG .
          docker push $REGISTRY/rocketseat-ci:$TAG
          docker tag $REGISTRY/rocketseat-ci:$TAG $REGISTRY/rocketseat-ci:latest
          docker push $REGISTRY/rocketseat-ci:latest
          IMAGE=$(echo $REGISTRY/rocketseat-ci:$TAG)
          echo "image=$IMAGE" >> $GITHUB_OUTPUT

      - name: Deploy to App Runner
        id: deploy-apprunner
        uses: awslabs/amazon-app-runner-deploy@main
        with:
          service: rocketseat-api
          image: ${{ steps.build-docker-image.outputs.image }}
          access-role-arn : arn:aws:iam::***************:role/app-runner-role
          region: us-east-1
          cpu : 1
          memory : 2
          port: 3000
          wait-for-service-stability-seconds: 180

      - name: App Runner Check
        run: echo "App Runner running... ${{ steps.deploy-apprunner.outputs.service-url }}"
```

- **name**:  `Semantic Release `:

    Nome da etapa que faz o lançamento semântico.

    `uses: cycjimmy/semantic-release-action@v4 `:
    Utiliza a ação semantic-release-action para fazer um release automatizado.

    `env: `:
    Define variáveis de ambiente para a etapa.

    `GITHUB_TOKEN: ${{ secrets.GH_TOKEN_RELEASE }} `:
    Utiliza um token de autenticação armazenado nos segredos do repositório (GH_TOKEN_RELEASE) para permitir que o semantic-release interaja com o GitHub, como criar tags, releases e gerenciar changelogs.


#### Criando o Token `"GITHUB_TOKEN: ${{ secrets.GH_TOKEN_RELEASE }}"`

![](image/ReleseConfiguration/create-token-github-step1.png)
![](image/ReleseConfiguration/create-token-github-step2.png)
![](image/ReleseConfiguration/create-token-github-step3.png)
![](image/ReleseConfiguration/create-token-github-step4.png)
![](image/ReleseConfiguration/create-token-github-step5.png)
![](image/ReleseConfiguration/create-token-github-step6.png)
![](image/ReleseConfiguration/create-token-github-step7.png)

Agora no repositório da api vamos configurar o token no actions secrets

![](image/ReleseConfiguration/create-token-github-step9.png)

![](image/ReleseConfiguration/create-token-github-step10.png)
![](image/ReleseConfiguration/create-token-github-step11png)
![](image/ReleseConfiguration/create-token-github-step12.png)

#### Configurando os passos do release

Agora vamos criar na raiz do projeto um arquivo `.releaserc`

```hcl
{
  "branches": [
    "main"
  ],
  "plugins": [
    "@semantic-release/commit-analyzer",
    "@semantic-release/release-notes-generator",
    [
      "@semantic-release/changelog",
      {
        "changelogFile": "CHANGELOG.md"
      }
    ],
    [
      "@semantic-release/git",
      {
        "assets": [
          "docs/CHANGELOG.md",
          "package.json",
          "package-lock.json"
        ]
      }
    ],
    "@semantic-release/npm",
    "@semantic-release/github",
  ]
}

```

Apos a edição , vamos executar no terminal os comandos referentes ao pacote do `"semantic"`.

`npm i @semantic-release/commit-analyzer @semantic-release/release-notes-generator @semantic-release/changelog @semantic-release/git @semantic-release/npm @semantic-release/github -D`

## 2 Rodando pipeline
Agora para testar o fluxo de vercionamento, vamos criar uma nova branch.

no terminal execute os comando :

`git status`
`main + !`
`git checkout -b feature/configure-release/andre`
`feature/configure-release/andre + !`
`git status`
`git add .`
`git commit -m "new configure semrelease"`
`git push --set-upstream origin feature/configure-release/andre`

Agora é necessário abrir um `"Pull Request"` da barnch que criamos para a branch `main`.

![](image/ReleseConfiguration/pull-request-release.png)

![](image/ReleseConfiguration/created-pull-request-release.png)

![](image/ReleseConfiguration/merge-pull-request-release.png)

![](image/ReleseConfiguration/done-merge-pull-request-release.png)

Após aprovarmos o merge vai inicira o processo no `"Git Hub Actions"`.

![](image/ReleseConfiguration/error-cannot-push.png)

Como podemos ver, ocorreu um erro. Esse erro está relacionado a permissões contidas em nossa pipelime. Então vamos editar o nosso arquivo `ci.yml` com o seguinte trecho de código.


```hcl
permissions:
  id-token: write
  contents: write
```
`git add .`
` git commit -m "fix: fix content repository"`
`git push`

desta forma teremos que executar um novo Pull Request.

![](image/ReleseConfiguration/new-pull-request-fix.png)

![](image/ReleseConfiguration/new-pull-request-fix-itens.png)

![](image/ReleseConfiguration/new-pull-request-fix-itens-description.png)

![](image/ReleseConfiguration/fix-merge-pull-request.png)

![](image/ReleseConfiguration/exec-fix-merge-pull-request.png)

![](image/ReleseConfiguration/error-permission.png)


Como podemos ver, ocorreu um erro. Esse erro está relacionado a permissões contidas em nossa pipelime. Então vamos editar o nosso arquivo `ci.yml` com o seguinte trecho de código.


## 3 Corrigindo problemas de integração


```hcl
permissions:
  id-token: write
  contents: write
  issues : write
  pull-requests: write
```
E devemos configura outro ponto em nosso repositorio git .

![](image/ReleseConfiguration/config-release-git-hub-actions.png)

![](image/ReleseConfiguration/change-workflow-permissions.png)

Agora vamos validar o escopo do token .

![](image/ReleseConfiguration/token-roles.png)

![](image/ReleseConfiguration/token-roles-permissions.png)


vamos garantir que temos permissão em issues epull-request.

![](image/ReleseConfiguration/token-roles-permissions-edit-pr-no-access.png)

vamos alterar para read and wirte 
![](image/ReleseConfiguration/token-roles-permissions-edit-pr-access-ok.png)


agora vamos realizar um noco commit.

`git add .`
` git commit -m "fix: fix config token permission git pull-request access"`
`git push`

vamos realizar todos os passos de pull request e merge ...


![](image/ReleseConfiguration/result-release.png)


