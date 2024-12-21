
# Sumário

- [✨ Contexto inicial e problema](#contexto-inicial-e-problema)
  - [Cenário](#cenário)
  - [Problemas](#problemas)
- [✨ Esboço do problema](#esboço-do-problema)
- [✨ Principais componentes](#principais-componentes)
  - [Arquitetura do Kubernetes](#arquitetura-do-kubernetes)
- [✨ Como executar um Cluster Kubernetes](#como-executar-um-cluster-kubernetes)
- [✨ Configurando Ambiente](#configurando-ambiente)
- [✨ Configurando e criando nosso primeiro Cluster](#configurando-e-criando-nosso-primeiro-cluster)
- [✨ Subindo o nosso primeiro Container](#subindo-o-nosso-primeiro-container)
- [✨ Valor dos manifestos declarativos e namespaces](#valor-dos-manifestos-declarativos-e-namespaces)
- [✨ Acessando container dentro do cluster](#acessando-container-dentro-do-cluster)
- [✨ Problemas e próximos passos](#problemas-e-próximos-passos)
- [✨ Criando um ReplicaSet](#criando-um-replicaset)
- [✨ Criando um Deployment](#criando-um-deployment)
- [✨ Acessando Pods](#acessando-pods)
- [✨ Trabalhando com Estratégias De Deploy](#trabalhando-com-estratégias-de-deploy)
  - [RollingUpdate](#rollingupdate)
  - [Recreate](#recreate)
- [✨ Explorando Variável de Ambiente na Aplicação](#explorando-variável-de-ambiente-na-aplicação)
- [✨ Entendendo sobre o ConfigMap](#entendendo-sobre-o-configmap)
- [✨ Explorando o objeto Secret](#explorando-o-objeto-secret)

# Conhecendo o Kubernetes

## ✨ Contexto inicial e problema

### Cenário

- "Temos Uma aplicação executando em container e essa execução em container traz algumas preocupações (Problemas)."
- "Temos várias aplicações executando em container e essa execução em container traz algumas preocupações (Problemas)."

### Problemas

- "E se um container falhar na execução?"
- "E se precisarmos de vários containers executando a mesma aplicação?"
- "E se precisarmos de um fluxo elástico?"
- "E se forem várias aplicações com vários containers?"
- "E o controle de uso de recursos de container?"

## ✨ Esboço do problema

No cenário temos uma aplicação "A", a aplicação "A" está contida em um container.

![](image/Kubernetes/app-a-container-a.png)

Quando o container a é executado, ele necessita de recursos computacionais

![](image/Kubernetes/app-a-container-a-computational-resources.png)

Caso ocorra algum problema no container ,o mesmo vai parar a execução e logo não existe mais.

Quando não trabalhamos com orquestração é muito difícil entender os motivos da causa do problema.

Quando não trabalhamos com orquestração não temos a visibilidade de quando um determinado problema irá ocorrer.

Caso ocorra o problema como será resolvido?

Sem a orquestração a solução desses problemas é muito complexa!

Logo é muito importante que esse cenário seja automatizado.

O kubernetes entra no cenário para resolver esses problemas.

#### **`Conceitos Básicos do Kubernetes`**

**`Arquitetura`**

- "Cluster": Conjunto de máquinas (nós) que executam aplicações em contêineres. Um cluster é composto por um `Master Node` e vários `Worker Nodes`.
- "Node": Uma única máquina em execução em um cluster Kubernetes, pode ser física ou virtual.
- "Pod": Unidade básica de execução em Kubernetes. Um `Pod` pode conter um ou mais contêineres que compartilham recursos de rede e armazenamento.

[O restante do documento segue com os conteúdos carregados do documento original.]
