# Conhecendo o Kubernetes

## ✨ Contexto inicial e problema

### Cenário

- `"Temos Uma aplicação executando em container e essa execução em container traz algumas preocupações (Problemas)."`
- `"Temos várias aplicações executando em container e essa execução em container traz algumas preocupações (Problemas)."`

### Problemas

- `"E se um container falhar na execução?"`
- `"E se precisarmos de vários containers executando a mesma aplicação?"`
- `"E se precisarmos de um fluxo elástico?"`
- `"E se forem várias aplicações com vários containers?"`
- `"E o controle de uso de recursos de container?"`

## ✨ Esboço do problema

No cenário temos uma aplicação `"A"`, a aplicação `"A"` está contida em um container.

![](image/Kubernetes/app-a-container-a.png)

Quando o container a é executado, ele necessita de recursos computacionais

![](image/Kubernetes/app-a-container-a-computational-resources.png)

Caso ocorra algum problema no container ,o mesmo vai parar a execução e logo não existe mais.

Quando não trabalhamos com orquestração é muito difícil entender os motivos da causa do problema.

Quando não trabalhamos com orquestração não temos a visibilidade de quando um determinado problema irá ocorrer.

Caso ocorra o problema como será resolvido?

Sem a orquestração a solução desses problemas é muito complexa!

Logo é muito importante que esse cenário seja automatizado.

O kubernetes entra no senário para resolver esses problemas.

#### **`Conceitos Básicos do Kubernetes`**

**`Arquitetura`**

- `"Cluster"`: Conjunto de máquinas (nós) que executam aplicações em contêineres. Um cluster é composto por um `Master Node` e vários `Worker Nodes`.
- `"Node"`: Uma única máquina em execução em um cluster Kubernetes, pode ser física ou virtual.
- `"Pod"`: Unidade básica de execução em Kubernetes. Um `Pod` pode conter um ou mais contêineres que compartilham recursos de rede e armazenamento.


**`Objetos Principais`**

- `"Pod"`: O menor objeto implantável no Kubernetes. Ele encapsula um ou mais contêineres, com armazenamento compartilhado e configurações de rede.
- `"Deployment"`: Gerencia a criação e atualização de `Pods`. Facilita o controle de versões e rollback.
- `"Service"`: Define uma maneira de expor uma aplicação em execução como um serviço de rede.
- `"ConfigMap"`: Armazena dados de configuração em pares chave-valor que podem ser consumidos por contêineres.
- `"Secret"`: Semelhante ao `ConfigMap`, mas usado para armazenar dados sensíveis, como senhas e chaves de API.


![](image/Kubernetes/app-a-container-a-computational-resources-sh.png)

**`"Self-Healing no Kubernetes"`**

O **self-healing** no Kubernetes é um recurso que permite ao cluster detectar e corrigir automaticamente falhas em seus componentes, garantindo maior disponibilidade e resiliência das aplicações.

**`"Como funciona"`**
- **Pods com falhas**: Se um Pod falhar (parar de responder ou entrar em estado de erro), o Kubernetes recria ou substitui automaticamente o Pod para restaurar o estado desejado.
- **Nodes indisponíveis**: Se um nó (node) falhar, os Pods que estavam nele são redistribuídos para outros nós disponíveis.
- **Checagens de saúde**: Kubernetes utiliza *liveness probes* e *readiness probes* para monitorar continuamente os Pods e determinar se estão saudáveis.

**`"Para que serve"`**
1. **Garantir alta disponibilidade**: Minimiza o tempo de inatividade ao corrigir falhas rapidamente.
2. **Automatizar recuperação**: Evita a necessidade de intervenção manual para problemas comuns.
3. **Melhorar resiliência**: Ajuda a manter o sistema funcionando mesmo em condições adversas.

Este recurso é essencial para aplicações modernas que demandam escalabilidade e alta confiabilidade.

![](image/Kubernetes/app-a-container-a-all.png)

Nesse mecanismo e chamado de hpa (Horizontal Pod Autoscaler)


#### O que é Horizontal Pod Autoscaler (HPA)?

O **Horizontal Pod Autoscaler (HPA)** é um recurso do Kubernetes que ajusta automaticamente o número de réplicas de pods em um deployment, replica set ou stateful set com base na utilização de métricas observadas (como uso de CPU, memória ou métricas personalizadas). 

#### Para que serve?

O HPA é usado para:

- **Escalabilidade automática**: Garante que o número de pods atenda à demanda variável do aplicativo.
- **Eficiência de recursos**: Ajuda a evitar subutilização ou sobrecarga de recursos.
- **Alto desempenho**: Mantém o desempenho do aplicativo em momentos de picos de uso.
- **Otimização de custos**: Ajusta a quantidade de recursos usados para evitar desperdício.

#### Como funciona?

1. Monitora métricas definidas, como uso de CPU/memória ou métricas personalizadas através do API Metrics Server.
2. Calcula o número ideal de réplicas com base nas metas configuradas.
3. Escala automaticamente o número de pods para atingir as metas.

Exemplo de uso comum:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-a
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app-a
  minReplicas: 3
  maxReplicas: 8
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

## ✨ Principais componentes

#### Arquitetura do Kubernetes

![](image/Kubernetes/arq-kubernetes.png)

- **Control Plane**:
O **Control Plane** gerencia o estado desejado do cluster, monitorando e controlando os componentes do Kubernetes. Ele coordena os nós, agendando workloads e respondendo a eventos do sistema.

---

- **API Server**:
O **API Server** é o ponto de entrada para todas as interações externas e internas com o Kubernetes. Ele expõe a API RESTful do Kubernetes e é responsável por autenticar e validar as solicitações.

---

- **Cloud Controller Manager**:
O **Cloud Controller Manager** integra o Kubernetes ao provedor de nuvem. Ele gerencia recursos dependentes da infraestrutura da nuvem, como balanceadores de carga, volumes de armazenamento e nós virtuais.

---

- **ETCD (Persistence Store)**:
O **ETCD** é o armazenamento de dados distribuído usado pelo Kubernetes para salvar todos os dados do cluster. Ele é a única fonte de verdade, contendo configurações, estados de objetos e informações críticas.

---

- **Kubelet**:
O **Kubelet** é o agente que roda em cada nó do cluster. Ele recebe instruções do API Server e garante que os containers estejam rodando como especificado nos objetos Pod.

---

- **Kube-proxy**:
O **Kube-proxy** gerencia as regras de rede dentro do cluster, garantindo a comunicação entre Pods e serviços, além de lidar com o balanceamento de carga interno.

---

- **Scheduler**:
O **Scheduler** decide em qual nó os Pods recém-criados devem ser executados, baseando-se em fatores como recursos disponíveis, restrições de afinidade e anti-afinidade.

---

- **Node**:
O **Node** é uma máquina (física ou virtual) que executa os workloads do Kubernetes. Cada nó contém um **Kubelet**, um **Kube-proxy** e o ambiente necessário para rodar containers.


## ✨ Como executar um Cluster Kubernetes

Tipos de cluster

- Gerenciado : Pago [AWS-EKS, GCP-GKE, AZURE-AKS]
- Não Gerenciado : OnPremise, (na sua máquina)

## ✨ Configurando Ambiente

Ferramentas:

- Kubectl: CLI 
- lens (IDE)
- K9s 
- (na sua máquina) : Kind(`"Kubernetes in docker"`), Minikube, K3d/k3s

https://kubernetes.io/pt-br/docs/tasks/tools/

https://docs.k8slens.dev/

## ✨ Configurando e criando nosso primeiro Cluster

- Vamos criar uma cluster, para isso vamos digitar o seguinte comando no terminal:

```
kind create cluster --name=first-cluster-rocketseat
```

Logo apos no terminal e apresentado o resukltado da criação:
```hcl

ster --name=first-cluster-rocketseat
Creating cluster "first-cluster-rocketseat" ...
 • Ensuring node image (kindest/node:v1.31.2) 🖼  ...
 ✓ Ensuring node image (kindest/node:v1.31.2) 🖼
 • Preparing nodes 📦   ...
 ✓ Preparing nodes 📦
 • Writing configuration 📜  ...
 ✓ Writing configuration 📜
 • Starting control-plane 🕹️  ...
 ✓ Starting control-plane 🕹️
 • Installing CNI 🔌  ...
 ✓ Installing CNI 🔌
 • Installing StorageClass 💾  ...
 ✓ Installing StorageClass 💾
Set kubectl context to "kind-first-cluster-rocketseat"
You can now use your cluster with:

kubectl cluster-info --context kind-first-cluster-rocketseat

Not sure what to do next? 😅  Check out https://kind.sigs.k8s.io/docs/user/quick-start/

```

O result acima indicou a o uso do contexto : ```kubectl cluster-info --context kind-first-cluster-rocketseat```

```hcl

PS C:\Repo\k8s\first-cluster> kubectl cluster-info --context kind-first-cluster-rocketseat
Kubernetes control plane is running at https://127.0.0.1:49448
CoreDNS is running at https://127.0.0.1:49448/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

```

Agora vamos rtoadr o comando `kubectl` que apresentará a lista de coamndo que podemos usar:

```hcl

 kubectl 
kubectl controls the Kubernetes cluster manager.

 Find more information at: https://kubernetes.io/docs/reference/kubectl/

Basic Commands (Beginner):
  create          Create a resource from a file or from stdin
  expose          Take a replication controller, service, deployment or pod and expose it as a new Kubernetes service
  run             Run a particular image on the cluster
  set             Set specific features on objects

Basic Commands (Intermediate):
  explain         Get documentation for a resource
  get             Display one or many resources
  edit            Edit a resource on the server
  delete          Delete resources by file names, stdin, resources and names, or by resources and label selector

Deploy Commands:
  rollout         Manage the rollout of a resource
  scale           Set a new size for a deployment, replica set, or replication controller
  autoscale       Auto-scale a deployment, replica set, stateful set, or replication controller

Cluster Management Commands:
  certificate     Modify certificate resources
  cluster-info    Display cluster information
  top             Display resource (CPU/memory) usage
  cordon          Mark node as unschedulable
  uncordon        Mark node as schedulable
  drain           Drain node in preparation for maintenance
  taint           Update the taints on one or more nodes

Troubleshooting and Debugging Commands:
  describe        Show details of a specific resource or group of resources
  logs            Print the logs for a container in a pod
  attach          Attach to a running container
  exec            Execute a command in a container
  port-forward    Forward one or more local ports to a pod
  proxy           Run a proxy to the Kubernetes API server
  cp              Copy files and directories to and from containers
  auth            Inspect authorization
  debug           Create debugging sessions for troubleshooting workloads and nodes
  events          List events

Advanced Commands:
  diff            Diff the live version against a would-be applied version
  apply           Apply a configuration to a resource by file name or stdin
  patch           Update fields of a resource
  replace         Replace a resource by file name or stdin
  wait            Experimental: Wait for a specific condition on one or many resources
  kustomize       Build a kustomization target from a directory or URL

Settings Commands:
  label           Update the labels on a resource
  annotate        Update the annotations on a resource
  completion      Output shell completion code for the specified shell (bash, zsh, fish, or powershell)

Subcommands provided by plugins:

Other Commands:
  api-resources   Print the supported API resources on the server
  api-versions    Print the supported API versions on the server, in the form of "group/version"
  config          Modify kubeconfig files
  plugin          Provides utilities for interacting with plugins
  version         Print the client and server version information

Usage:
  kubectl [flags] [options]

Use "kubectl <command> --help" for more information about a given command.
Use "kubectl options" for a list of global command-line options (applies to all commands).

```

Logo vamos usar o seguinte comando : `kubectl get nodes`


```
PS C:\Repo\k8s\first-cluster> kubectl get nodes
NAME                                     STATUS   ROLES           AGE   VERSION
first-cluster-rocketseat-control-plane   Ready    control-plane   16m   v1.31.2
PS C:\Repo\k8s\first-cluster> 
```

Agora vamos abiri o lens.

![](image/Kubernetes/start-lens.png)


## ✨ Configurando Cluster com multiplos nós

Vamos Excluir o primeiro cluster : `kind delete cluster --name=first-cluster-rocketseat`

Agora vamos escrever um script manifesto para a criação de um novo cluster.

- `"kind.yaml"`: vamos criar um novo arquivo para incluir as intruções do novo cluster

- `"script base"`: Inclua o seguinte código

```hcl

kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
```

- `"Command"` : No terminal digite o seguinte comando :

`kind create cluster --config kind.yaml --name=secund-cluster-rocketseat` 

![](image/Kubernetes/create-secund-cluster-rocketseat.png)

`kubectl cluster-info --context kind-secund-cluster-rocketseat`

`kubectl get nodes`

![](image/Kubernetes/get-nodes-secund-cluster-rocketseat.png)

#### Explicação do Arquivo de Configuração Kind

Este é um arquivo de configuração para o Kubernetes in Docker (Kind), utilizado para criar clusters Kubernetes localmente com fins de desenvolvimento e teste.

---

##### **Cabeçalho**

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
```

1. **`kind: Cluster`**
   - Especifica o tipo de recurso que o arquivo define.
   - Indica que o arquivo descreve um *cluster* completo do Kubernetes que será gerenciado pelo Kind.

2. **`apiVersion: kind.x-k8s.io/v1alpha4`**
   - Define a versão da API do Kind utilizada neste arquivo.
   - Esta versão controla as opções de configuração disponíveis para criar e gerenciar clusters.

---

##### **Configuração de Nós**

```yaml
nodes:
  - role: control-plane
  - role: worker
```

1. **`nodes`**
   - Define os nós (*nodes*) que farão parte do cluster.
   - Cada nó é uma instância do Kubernetes e pode ser configurado como um nó de controle ou de trabalho.

2. **`- role: control-plane`**
   - Define que o primeiro nó será um **nó de controle** (*control-plane*).
   - O nó de controle é responsável por executar os componentes principais do Kubernetes, incluindo:
     - **API Server**: Ponto de entrada para comandos e consultas ao cluster.
     - **Controller Manager**: Garante que o estado desejado dos objetos seja mantido.
     - **Scheduler**: Aloca pods em nós disponíveis.
     - **etcd**: Banco de dados chave-valor que armazena todos os dados do cluster.

3. **`- role: worker`**
   - Define que o segundo nó será um **nó de trabalho** (*worker*).
   - Os nós de trabalho executam os **pods**, que contêm os contêineres das aplicações.
   - Cada nó de trabalho é gerenciado pelo nó de controle e executa:
     - **kubelet**: Agente que garante que os contêineres dos pods sejam executados como especificado.
     - **kube-proxy**: Mantém regras de rede para a comunicação interna e externa do cluster.
     - **Container Runtime**: Software que executa os contêineres (ex.: Docker, containerd).

---

##### **Resumo**

Este arquivo cria um cluster Kubernetes com:

- **1 Nó de Controle**: Responsável por gerenciar o cluster.
- **1 Nó de Trabalho**: Responsável por executar os aplicativos em contêiner.

Para criar o cluster utilizando este arquivo, use o comando:

```bash
kind create cluster --config=<nome-do-arquivo.yaml>
```

Esse tipo de configuração é ideal para criar ambientes simples e locais, simulando clusters Kubernetes reais

![](image/Kubernetes/view-docker-creted-cluster.png)




