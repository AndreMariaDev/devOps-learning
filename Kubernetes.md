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




# Orquestrando Containers


## ✨ Subindo o nosso primeiro Container

Aqui vamos iniciar a creação de um pod 

Como um Pod é responsável por (Rodar a nossa aplicação dentro do cluster)
Vamos trabalhar nesse ponto.


- `"imagem para o container"` : Vamos acessar o site do dockerhub https://hub.docker.com/_/nginx/tags?name=alpine


- `kubectl run nginx --image=nginx:stable-alpine3.20-perl`

- `kubectl get pods`

- `kubectl describe pod nginx`


![](image/Kubernetes/pod-nginx.png)

- `"Execução de forma IMPERATIVA"` : `kubectl run nginx --image=nginx:stable-alpine3.20-perl`

- `"Não temos o controlador"`

## ✨ Valor dos manifestos declarativos e namespaces

Vamos deletar o exemplo: ` kubectl delete pod nginx`

Agora vamos criar a namespace : `kubectl create namespace first-app`

Feito isso vamos criar o pod via arquivo. Para isso vamos criar um novo arquivo pod.yaml

#### Explicação Detalhada do Script YAML para Criação de um Pod no Kubernetes

Este documento explica cada passo do script YAML fornecido para criar um Pod no Kubernetes.

```yaml
apiVersion: v1
kind: Pod

metadata:
  name: nginx

spec:
  containers:
    - name : nginx
      image: nginx:stable-alpine3.20-perl
      ports:
        - containerPort: 80
      resources:
        requests:
          cpu: 100m
          memory: 64Mi
        limits:
          cpu: 200m
          memory: 128Mi
```

####  1. **Cabeçalho do Documento**

#### `apiVersion: v1`
- Define a versão da API do Kubernetes utilizada para este recurso.
- `v1` é a versão estável para recursos básicos como `Pod`.

#### `kind: Pod`
- Especifica o tipo de recurso que será criado.
- Neste caso, estamos criando um **Pod**, que é a menor unidade executável no Kubernetes.

---

#### 2. **Metadados do Pod**

#### `metadata:`
- Define informações adicionais sobre o recurso.

#### `name: nginx`
- Atribui o nome **nginx** ao Pod.
- Este nome deve ser único dentro do namespace onde o Pod está sendo criado.

---

#### 3. **Especificações do Pod**

#### `spec:`
- Contém a descrição detalhada de como o Pod será configurado.

#### `containers:`
- Define os containers que serão executados no Pod. Cada Pod pode conter um ou mais containers.

##### `- name: nginx`
- Especifica o nome do container.
- O nome deve ser único dentro do Pod e é usado para identificar o container.

##### `image: nginx:stable-alpine3.20-perl`
- Indica a imagem Docker que será usada para criar o container.
- Neste caso, a imagem é **nginx:stable-alpine3.20-perl**, que é uma versão leve e estável do servidor web NGINX.

##### `ports:`
- Define as portas expostas pelo container.

###### `- containerPort: 80`
- Especifica que o container exporá a porta **80**, comumente usada para serviços HTTP.

---

#### 4. **Recursos do Container**

#### `resources:`
- Configura os limites e as solicitações de recursos para o container.

#### `requests:`
- Representa a quantidade mínima de recursos garantidos para o container.

##### `cpu: 100m`
- O container requer pelo menos **100 millicores** (0,1 vCPU).

##### `memory: 64Mi`
- O container requer pelo menos **64 MiB** de memória.

#### `limits:`
- Representa a quantidade máxima de recursos que o container pode usar.

##### `cpu: 200m`
- O container pode usar até **200 millicores** (0,2 vCPU).

##### `memory: 128Mi`
- O container pode usar até **128 MiB** de memória.

---

#### Resumo
Este script YAML configura um Pod com:
- Um container baseado na imagem **nginx:stable-alpine3.20-perl**.
- Exposição da porta 80.
- Garantia de uso mínimo de **0,1 vCPU** e **64 MiB** de memória.
- Limite máximo de **0,2 vCPU** e **128 MiB** de memória.

O Pod criado é eficiente para serviços de baixa utilização, ideal para ambientes leves ou de desenvolvimento.

Agora vamos rodar o cmando :
´kubectl apply -f pod.yaml -n first-app´

#### Comando `kubectl apply -f pod.yaml -n first-app`

Este comando utiliza o `kubectl` para aplicar configurações de um arquivo YAML a um cluster Kubernetes. Aqui estão os detalhes de cada parte do comando:

#### 1. `kubectl`
- **Definição**: `kubectl` é a ferramenta de linha de comando utilizada para interagir com o Kubernetes.
- **Função neste comando**: Serve como a base para executar comandos que gerenciam recursos no cluster Kubernetes.

#### 2. `apply`
- **Definição**: O subcomando `apply` é usado para aplicar ou atualizar a configuração de recursos no cluster.
- **Função neste comando**: Indica ao Kubernetes que ele deve criar ou atualizar os recursos descritos no arquivo `pod.yaml`.

#### 3. `-f pod.yaml`
- **Definição**:
  - O argumento `-f` (abreviação de `--filename`) especifica o arquivo YAML contendo a definição dos recursos Kubernetes a serem aplicados.
  - `pod.yaml` é o nome do arquivo que contém a descrição dos recursos, como pods, services ou deployments.
- **Função neste comando**: Fornece ao `kubectl` os detalhes da configuração do pod para aplicação no cluster.

#### 4. `-n first-app`
- **Definição**:
  - `-n` (abreviação de `--namespace`) especifica o namespace no qual os recursos devem ser criados ou atualizados.
  - `first-app` é o nome do namespace alvo.
- **Função neste comando**: Garante que o recurso seja aplicado no namespace correto (neste caso, `first-app`).
  - Se o namespace não for especificado, o Kubernetes utiliza o namespace padrão (geralmente chamado `default`).

---

Com isso, o comando `kubectl apply -f pod.yaml -n first-app` permite criar ou atualizar recursos em um namespace Kubernetes de maneira organizada e eficiente.


![](image/Kubernetes/pod-first-app.png)


## ✨ Acessando container dentro do cluster

#### Explicação do Comando `kubectl port-forward pod/nginx -n first-app 8080:80`

O comando `kubectl port-forward` é usado no Kubernetes para encaminhar portas de um recurso (como um pod) no cluster para o ambiente local. Isso permite acessar aplicativos executados no cluster sem expô-los publicamente por meio de um serviço ou ingress.

#### Detalhamento do comando:

```bash
kubectl port-forward pod/nginx -n first-app 8080:80
```

#### Componentes do Comando:

1. **`kubectl port-forward`**:
   - Essa é a funcionalidade do `kubectl` que inicia o encaminhamento de portas.

2. **`pod/nginx`**:
   - Especifica o recurso que será o alvo do encaminhamento de portas.
   - Aqui, estamos nos referindo ao pod chamado `nginx`. O prefixo `pod/` indica explicitamente que o recurso é um pod.

3. **`-n first-app`**:
   - Especifica o namespace onde o pod está localizado.
   - Neste caso, o pod `nginx` pertence ao namespace `first-app`.

4. **`8080:80`**:
   - Define o mapeamento de portas.
   - O primeiro número (`8080`) é a porta na máquina local (host), que será usada para acessar o aplicativo.
   - O segundo número (`80`) é a porta no pod, onde o aplicativo está escutando.

#### O que acontece ao executar esse comando?

- Uma conexão é estabelecida entre o pod `nginx` no namespace `first-app` e a máquina local.
- As solicitações enviadas para `localhost:8080` (máquina local) serão encaminhadas para a porta `80` no pod `nginx` dentro do cluster Kubernetes.
- Não é necessário expor o serviço ou criar um recurso adicional para acessar o aplicativo.

#### Cenários de Uso

- **Depuração**:
  - Verificar se o aplicativo está funcionando corretamente no pod.
- **Desenvolvimento Local**:
  - Acessar aplicativos no cluster sem configurar serviços ou ingress.
- **Teste Temporário**:
  - Testar endpoints ou funcionalidades de um aplicativo diretamente.

#### Observações Importantes

1. **Necessidade de Acesso ao Cluster**:
   - O comando requer que você tenha permissões para acessar o namespace e o pod especificados.

2. **Conexão Limitada ao Tempo de Execução**:
   - O encaminhamento permanece ativo apenas enquanto o comando estiver em execução no terminal.

3. **Desempenho e Escalabilidade**:
   - Este método é ideal para uso em casos simples ou temporários. Para ambientes de produção, é recomendado usar serviços, ingress ou outras soluções mais robustas.

4. **Restrições de Portas Locais**:
   - Certifique-se de que a porta local especificada (`8080`) não esteja em uso por outro processo.

---

### Exemplo Prático

Após executar o comando:

```bash
kubectl port-forward pod/nginx -n first-app 8080:80
```

Você pode acessar o aplicativo executado no pod `nginx` digitando o seguinte no navegador ou usando ferramentas como `curl`:

```bash
http://localhost:8080
```

Isso redirecionará as solicitações para o pod `nginx` na porta `80` dentro do namespace `first-app` do cluster Kubernetes.



## ✨ Problemas e próximos passos

#### O Que Acontece ao deleta o `Pod` criados?

Ao deletar o Pod:

1. **`"Recriação Automática"`**: Não ocorrerá recriação automática porque o Pod foi criado diretamente sem estar associado a um controlador.
2. **`"Perda Permanente"`**: Uma vez deletado, o Pod e qualquer dado nele contido (se não houver volumes persistentes configurados) serão permanentemente removidos.

Usar um `"ReplicaSet"` poderia ser uma solução paliativa para garantir a recriação automática de um Pod caso ele seja excluído. 

#### O Que Acontece ao Deletar os Pods Criados por um ReplicaSet?

Se você deletar os pods criados por um **ReplicaSet** no Kubernetes, o ReplicaSet automaticamente recriará os pods para garantir que o número especificado em `spec.replicas` seja mantido. 

Esse comportamento faz parte do objetivo do ReplicaSet de assegurar que o estado desejado do sistema seja preservado.

---

#### Explicação Detalhada

#### 1. **ReplicaSet Monitora o Estado**
O ReplicaSet verifica constantemente o número de pods em execução, baseando-se no `selector.matchLabels`. 

- Quando detecta que há menos pods do que o especificado em `spec.replicas`, ele cria novos pods para restaurar o estado desejado.

---

#### 2. **Ação Após a Deleção**
- Ao deletar um pod gerenciado pelo ReplicaSet, ele detecta a ausência desse pod e recria um novo imediatamente.
- O novo pod será baseado no modelo definido na seção `template` do ReplicaSet.

---

#### 3. **Imutabilidade dos Pods**
- Os pods criados pelo ReplicaSet são independentes e imutáveis. Alterar diretamente um pod (como modificar sua configuração) não afeta o modelo do ReplicaSet.
- Se um pod for modificado manualmente, o ReplicaSet substituirá esse pod por outro que siga o modelo original.

---

#### 4. **Impacto no Cluster**
- Caso todos os pods gerenciados sejam deletados, o ReplicaSet recriará os pods até atingir o número especificado em `replicas`.
- Se os recursos do cluster (CPU, memória, etc.) forem insuficientes, o ReplicaSet continuará tentando recriar os pods até que os recursos estejam disponíveis.

---

#### Exceção: Deleção do Próprio ReplicaSet
- Se o **ReplicaSet** for deletado, todos os pods gerenciados por ele também serão removidos, já que dependem do ReplicaSet para existir.

---

#### Caso Prático
Você pode testar a recriação automática dos pods com os seguintes comandos:

1. **Deletar um Pod**:
   ```bash
   kubectl delete pod <nome-do-pod>
   ```


2. **Verificar os Pods Disponíveis**:
    ```bash
    kubectl get pods
    ```
    Após a execução, você verá que um novo pod foi recriado automaticamente, com um nome diferente, mas baseado no modelo definido no ReplicaSet.

#### Resumo

  - Manutenção do número de réplicas: O ReplicaSet garante que o número de réplicas definido seja sempre mantido.

  - Recriação automática: Pods deletados serão recriados automaticamente.

  - Imutabilidade: Alterar pods diretamente não afeta o modelo do ReplicaSet; eles serão substituídos.

  - Deleção do ReplicaSet: Se o ReplicaSet for deletado, os pods associados também serão removidos.

  Este comportamento torna o ReplicaSet uma ferramenta poderosa para manter a alta disponibilidade de aplicativos no Kubernetes.

## ✨ Criando um ReplicaSet

#### Usando o ReplicaSet
```yaml
apiVersion: apps/v1
kind: ReplicaSet

metadata:
  name: nginx

spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec: 
      containers:
        - name : nginx
          image: nginx:stable-alpine3.20-perl
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 100m
              memory: 64Mi
            limits:
              cpu: 200m
              memory: 128Mi
```

#### Explicação do Comando ReplicaSet

O comando especificado é um manifesto em formato YAML usado para criar um **`"ReplicaSet"`** no Kubernetes. 

Um **`"ReplicaSet"`** é responsável por garantir que um número especificado de réplicas **`"(pods)"`**  esteja sempre em execução. 

Abaixo está a explicação detalhada de cada seção:

#### Estrutura do Manifesto

#### **`"apiVersion: apps/v1"`** 
Define a versão da API utilizada para o recurso. Nesse caso, o ReplicaSet usa a API `apps/v1`, que é estável para gerenciar aplicativos no Kubernetes.

---

#### **`"kind: ReplicaSet"`** 
Especifica o tipo de recurso que está sendo criado. Aqui, o recurso é um **ReplicaSet**.

---

#### **`"metadata:"`** 
Contém informações básicas de identificação do objeto.

- **name**: O nome do ReplicaSet. No exemplo, ele é chamado de `nginx`.

---

#### **`"spec:"`** 
Define a especificação desejada para o ReplicaSet, como o número de réplicas e a configuração dos pods.

#### **`"replicas: 5"`** 
Especifica o número desejado de réplicas do pod. O ReplicaSet garante que exatamente 5 pods estejam sempre em execução.

#### **`"selector:"`** 
Define como o ReplicaSet identifica os pods que ele deve gerenciar.

- **`matchLabels`**: Especifica os rótulos usados para associar os pods ao ReplicaSet. Neste caso, ele seleciona pods com o rótulo `app: nginx`.

#### **`"template:"`** 
Define o modelo (template) para os pods que o ReplicaSet irá criar. Essa seção é usada para configurar os pods.

- **`metadata`**:
  - **`labels`**: Define os rótulos atribuídos aos pods criados. Esses rótulos devem corresponder aos definidos em `selector.matchLabels`.

- **`spec`**: 
  Configurações específicas dos containers dentro dos pods.

#### **`"containers:"`** 
Lista os containers que devem ser executados no pod.

- **`name`**: Nome do container (no exemplo, `nginx`).
- **`image`**: Imagem do container a ser usada. Aqui, é especificada a imagem `nginx:stable-alpine3.20-perl`, que é uma versão leve do NGINX com suporte ao Perl.
- **`ports`**:
  - **`containerPort`**: Define a porta exposta pelo container. No exemplo, o container escuta na porta 80.

#### **`"resources:"`** 
Define as solicitações e limites de recursos para o container.

- **`requests`**: Recursos mínimos garantidos para o container.
  - **`cpu`**: `100m` (milicores de CPU) — o container receberá pelo menos 0,1 de um núcleo.
  - **`memory`**: `64Mi` — o container terá pelo menos 64 MiB de memória garantida.

- **`limits`**: Recursos máximos permitidos para o container.
  - **`cpu`**: `200m` (milicores de CPU) — o container não pode usar mais do que 0,2 de um núcleo.
  - **`memory`**: `128Mi` — o container não pode usar mais do que 128 MiB de memória.

---

#### Resumo
Este manifesto cria um **`"ReplicaSet"`** chamado `nginx`, que mantém 5 réplicas de um pod executando um container NGINX baseado na imagem `nginx:stable-alpine3.20-perl`. 
Os pods são configurados para escutar na porta 80 e possuem restrições de CPU e memória definidas para otimizar o uso de recursos do cluster.

Agora vamos executar o comando:

```bash
kubectl apply -f replicaset.yaml
```

![](image/Kubernetes/create-replicas.png)

![](image/Kubernetes/replicaset-pods.png)



## ✨ Problemas de um ReplicaSet

#### Problemas de Usar o ReplicaSet no Kubernetes

O objeto **`"ReplicaSet"`** no Kubernetes é responsável por garantir que um número especificado de réplicas de um pod esteja sempre em execução. No entanto, seu uso direto pode trazer alguns problemas e limitações, principalmente quando comparado a outros recursos, como o **`"Deployment**. Abaixo estão os principais problemas e implicações de usar o ReplicaSet no comando fornecido:

1. **`"Falta de Gerenciamento de Atualizações**
O **`"ReplicaSet"`"`** não fornece uma maneira nativa de gerenciar atualizações ou rollbacks. Isso significa que:

- Para atualizar uma imagem ou qualquer outra configuração, o ReplicaSet existente precisa ser deletado ou modificado manualmente, o que pode causar interrupções.
- Rollbacks em caso de falhas não são automáticos. Se algo der errado, é necessário reverter manualmente a configuração para um estado anterior.

Em contrapartida, o **`"Deployment"`** abstrai o ReplicaSet e oferece suporte nativo para atualizações declarativas e seguras.

2. **`"Complexidade no Escalonamento Manual"`** 
Embora o ReplicaSet permita o escalonamento ajustando o campo `replicas`, o processo é manual e sujeito a erros. Ferramentas de escalonamento automático (Horizontal Pod Autoscaler) geralmente são configuradas em conjunto com um Deployment e não diretamente com um ReplicaSet.

3. **`"Gestão Limitada de Ciclo de Vida"`** 
O ReplicaSet não controla o ciclo de vida completo de um aplicativo. Ele apenas mantém a quantidade de réplicas definida, mas não gerencia estados intermediários, como:

- Estratégias de atualização (por exemplo, `RollingUpdate` ou `Recreate`).
- Orquestração em cenários de alta complexidade (como blue/green deployments ou canary deployments).

4. **`"Aumento do Risco de Configurações Incorretas"`** 
Sem a abstração e automação fornecidas por um Deployment:

- Configurações podem ser aplicadas de maneira inconsistente.
- Existe um risco maior de deletar ou alterar acidentalmente o ReplicaSet em produção, causando interrupções.

5. **`"Menor Adoção e Suporte"`** 
Na prática, o uso direto de ReplicaSets é raro, pois o Deployment é amplamente adotado devido à sua flexibilidade e recursos adicionais. Isso pode resultar em:

- Menor suporte de comunidades e ferramentas de terceiros.
- Dificuldades para integrar configurações de CI/CD e práticas modernas de DevOps diretamente com ReplicaSets.

6. **`"Observabilidade Reduzida"`** 
Enquanto um Deployment oferece melhores métricas e status para acompanhar o progresso de atualizações e escalonamento, o ReplicaSet não oferece esses níveis de observabilidade de maneira integrada.

#### Considerações Finais
Embora o ReplicaSet seja a base para recursos como o Deployment, seu uso direto é recomendado apenas para casos muito específicos, como:

- Cenários onde não é necessário realizar atualizações no aplicativo.
- Configurações extremamente simples e estáveis que não exigem a abstração e automação oferecidas por um Deployment.

Na maioria dos casos, é mais adequado usar um **`"Deployment"`** , que oferece um gerenciamento mais robusto, eficiente e seguro para aplicações no Kubernetes.

**`"LOGO O REPLICASET NÃO É UM OBJETOS DE IMPLANTAÇÃO MAS SIM UMA CONTROLADOR DE NÚMERO DE PODS. ELE NÃO TEM SUPORTE A TROCA DE TAGS."`**

## ✨ Criando um Deployment

Agora para resolver o problema visto no uso do **`"ReplicaSet"`** vamos criar um novo arquivo `deployment.yaml`

Após a criação do arquivo vamos preencher com o seguinte script:

```yaml

apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx

spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:stable-alpine3.20-perl
        resources:
          requests:
            cpu: 100m
            memory: 64Mi
          limits:
            memory: "128Mi"
            cpu: "200m"
        ports:
        - containerPort: 80


```

```bash
kubectl apply -f deployment.yaml -n first-app
```

![](image/Kubernetes/overview-deployment-pods.png)

Vimos que a versão da imagem é `nginx:stable-alpine3.20-perl`
![](image/Kubernetes/overview-deployment-pod-version.png)


Para testar o ideia de troca de versão vamos editar o arquivo `deployment.yaml` substituindo `nginx:stable-alpine3.20-perl` para `nginx:mainline-alpine3.20-slim`

```yaml

apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx

spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:mainline-alpine3.20-slim
        resources:
          requests:
            cpu: 100m
            memory: 64Mi
          limits:
            memory: "128Mi"
            cpu: "200m"
        ports:
        - containerPort: 80


```

![](image/Kubernetes/overview-deployment-pod-version-change.png)


## ✨ Acessando Pods

Componente da interface de rede onde os acessos pelo Client é distribuido entre os pods.

Agora para resolver o problema de acesso vamos criar um novo arquivo `service.yaml`

Após a criação do arquivo vamos preencher com o seguinte script:

```yaml

apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
spec:
  type: ClusterIP
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80 # - containerPort: 80 deployment.yaml


```

### Explicação Detalhada do Script YAML

O script abaixo descreve um recurso `Service` no Kubernetes, que atua como uma abstração para expor aplicações executadas em um conjunto de pods.

#### Linha a Linha do Script

#### Cabeçalho do Documento

```yaml
apiVersion: v1
```
- **apiVersion**: Define a versão da API Kubernetes utilizada para este recurso. Aqui, `v1` é a versão para recursos básicos como `Service`.

```yaml
kind: Service
```
- **kind**: Especifica o tipo de recurso Kubernetes. Neste caso, é um `Service`.

```yaml
metadata:
  name: nginx-svc
```
- **metadata**: Contém informações adicionais sobre o recurso.
  - **name**: Define o nome do `Service`, neste caso, `nginx-svc`.

### Especificações do Service

```yaml
spec:
```
- **spec**: Define a configuração detalhada do recurso. Tudo que está dentro de `spec` refere-se à implementação do `Service`.

```yaml
  type: ClusterIP
```
- **type**: Define como o `Service` será exposto.
  - **ClusterIP**: O padrão. Permite o acesso ao `Service` apenas dentro do cluster.

```yaml
  selector:
    app: nginx
```
- **selector**: Define os critérios para selecionar os pods associados ao `Service`.
  - **app: nginx**: Seleciona todos os pods com o rótulo `app: nginx`.

```yaml
  ports:
```
- **ports**: Define as portas expostas pelo `Service`.

#### Configuração de Portas

```yaml
  - port: 80
```
- **port**: Porta em que o `Service` estará escutando. Aqui, é a porta `80`.

```yaml
    targetPort: 80
```
- **targetPort**: Porta no container para onde o tráfego será redirecionado. Aqui, também é `80`.
  - Esta porta deve corresponder ao **containerPort** definido no manifesto do `Deployment` (por exemplo, `deployment.yaml`).

---

#### Resumo do Fluxo
1. **Criar o Service**:
   - O Kubernetes identifica que é um recurso do tipo `Service`.
2. **Filtrar Pods**:
   - Usa o seletor `app: nginx` para encontrar os pods associados.
3. **Redirecionar o Tráfego**:
   - O tráfego enviado para o `Service` na porta `80` é redirecionado para a porta `80` nos containers selecionados.




```bash
kubectl apply -f service.yaml -n first-app
```

![](image/Kubernetes/service-view.png)


para acessar vamos rodar o seguinte comando:

```bash
kubectl port-forward svc/nginx-svc -n first-app 8080:80
```

### Explicação detalhada do comando
**Comando:**  
`kubectl port-forward svc/nginx-svc -n first-app 8080:80`

#### 1. **Contexto**  
Este comando é usado para criar um túnel local que encaminha o tráfego de uma porta no computador do usuário para uma porta em um serviço (Service) no cluster Kubernetes.

#### 2. **Componentes do comando**  

#### `kubectl`
É a ferramenta de linha de comando usada para interagir com clusters Kubernetes.  

#### `port-forward`
É um subcomando do `kubectl` que permite mapear uma porta do ambiente local para um pod ou serviço no cluster Kubernetes.  
- Com isso, você pode acessar recursos do cluster que normalmente não estariam expostos publicamente.  

#### `svc/nginx-svc`
Indica que o redirecionamento será feito para um **Service** chamado `nginx-svc`.  
- O prefixo `svc/` é usado para especificar que o alvo é um **Service**.  

#### `-n first-app`
Especifica o namespace onde o **Service** está localizado.  
- Neste caso, o namespace é `first-app`.  
- Se este argumento fosse omitido, o comando usaria o namespace padrão (`default`).  

#### `8080:80`
Define o mapeamento das portas:  
- `8080`: Porta local do computador onde o tráfego será recebido.  
- `80`: Porta do serviço dentro do cluster Kubernetes para a qual o tráfego será encaminhado.  
- O tráfego que chega em `localhost:8080` no ambiente local será redirecionado para o serviço `nginx-svc` na porta `80`.

---

#### 3. **Uso prático**  
Após executar este comando, você pode acessar o serviço `nginx-svc` diretamente no navegador ou com ferramentas como `curl` via:  

http://localhost:8080

#### 4. **Cenários de uso comuns**
- Testar um serviço no cluster sem configurá-lo como um recurso publicamente acessível.  
- Depuração ou desenvolvimento local, acessando serviços internos do Kubernetes de forma segura.  

---

#### 5. **Notas importantes**
- O comando precisa de acesso ao cluster Kubernetes configurado no `kubectl`.  
- Certifique-se de que a porta local (8080) não esteja em uso por outro processo antes de executar o comando.  
- Se o Service estiver configurado corretamente no cluster, o tráfego será redirecionado para os pods associados a ele.


Como vimos solucionamos o acesso interno aos pods , não não solucionamos o problema de acesso externo.
✨

# Explorando Deployment e cenários em uma aplicação real


## ✨ Conteinerizando a nossa aplicação

Vamos usar o DockerFile da Aplicação Exemplo da Sessão  Docker Tutorial.

## ✨ Criando os Objetos do Kubernetes

Vamos Utilizar o Docker Hub para trabalhar com as imagens da nossa aplicação

Desta forma é necessário fazer o login

```bash
docker login 
```

```yaml
# begin build
FROM node:18-alpine3.19 AS build

WORKDIR /usr/src/app

COPY package.json ./

RUN npm install

COPY . .

RUN npm run build

RUN npm ci --only=production
#end build

#begin excution
FROM node:18-alpine3.19

WORKDIR /usr/src/app

COPY --from=build /usr/src/app/package.json ./package.json
COPY --from=build /usr/src/app/dist ./dist
COPY --from=build /usr/src/app/node_modules ./node_modules

EXPOSE 3000

CMD ["npm", "run", "start:prod"]
#end excution
```

```bash
docker build -t api-rocket:v1 .
```

```bash
docker image ls api-rocket
```

```bash
docker tag api-rocket:v1 andremariadevops/api-rocket:v1 .
```

```bash
docker push andremariadevops/api-rocket:v1 
```

Agora vamos criar um objeto Deployment Kubernetes:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: api-rocket

spec:
  replicas: 2
  selector:
    matchLabels:
      api: api-rocket
  template:
    metadata:
      labels:
        api: api-rocket
    spec:
      containers:
      - name: api-rocket
        image: andremariadevops/api-rocket:v1
        resources:
          requests:
            cpu: 100m
            memory: 64Mi
          limits:
            cpu: "200m"
            memory: "128Mi"
        ports:
        - containerPort: 3000
```


![](image/Kubernetes/file-deployment-docker-hub.png)

```bash
kubectl create namespace ns-rocket
```

![](image/Kubernetes/commands-deployment.png)


```bash
kubectl apply -f k8s -n ns-rocket
```


![](image/Kubernetes/lens-view-pod-docker-hub.png)

## ✨ Criando Service e explorando imagePullPolicy

Agora vamos criar um novo arquivo `service.yaml`

Após a criação do arquivo vamos preencher com o seguinte script:

```yaml

apiVersion: v1
kind: Service
metadata:
  name: api-rocket-svc
spec:
  type: ClusterIP
  selector:
    api: api-rocket #  selector: matchLabels: api: api-rocket deployment.yaml
  ports:
  - port: 80
    targetPort: 3000 # - containerPort: 80 deployment.yaml


```

```bash

kubectl apply -f k8s/service.yaml -n ns-rocket

```

![](image/Kubernetes/commands-service.png)

Vamos incluir um novo end-point em nossa api :

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Rocketset Api!';
  }

  getExample(): string {
    return 'Estou rodando no K8s!';
  }
}
```

```typescript
import { Controller, Get } from '@nestjs/common';
import { AppService } from './app.service';

@Controller()
export class AppController {
  constructor(private readonly appService: AppService) {}

  @Get()
  getHello(): string {
    return this.appService.getHello();
  }

  @Get('/example-k8s')
  getExample(): string {
    return this.appService.getExample();
  }
}
```

NO cenário de CI/CD teriamos :

- 1 Commit 
- 2 Commit to build image
- 3 push
- 4 delevery

Como esse fluxo ainda não está automatizado vamos realizar manualmente os passos.

Mas antes vamos pensar sobre version e sua importância.

### Importância do Versionamento de Imagens Docker

- Por que versionar imagens Docker?

Manter o versionamento de imagens Docker é uma prática essencial para garantir a consistência, rastreabilidade e controle em ambientes de desenvolvimento, teste e produção. 

### Benefícios do Versionamento:

1. **Consistência entre Ambientes:**
   - Usar uma tag específica, como `v1.0.0` ou `1.2.3`, assegura que o ambiente de produção utilize exatamente a mesma versão de imagem que foi testada em desenvolvimento e homologação.
   - Evita problemas relacionados a mudanças inesperadas em imagens mais genéricas, como `latest`.

2. **Rastreabilidade:**
   - O versionamento permite identificar rapidamente qual versão da aplicação está em execução. Isso é fundamental para depuração, rollback e auditorias.
   - Sem versionamento, pode ser difícil ou impossível rastrear qual código ou configuração está sendo usado em um ambiente.

3. **Controle de Atualizações:**
   - Com imagens versionadas, atualizações podem ser planejadas e testadas cuidadosamente antes de serem aplicadas.
   - Reduz a probabilidade de interrupções ou falhas causadas por mudanças inesperadas.


### Conclusão

O versionamento de imagens Docker é crucial para garantir controle e estabilidade em ambientes de desenvolvimento e produção. A configuração do `"imagePullPolicy"`, por sua vez, regula como e quando as imagens devem ser atualizadas, complementando as boas práticas de versionamento e evitando problemas causados por mudanças inesperadas.



Logo vamos testar o cenário onde realizamos o `build` e `push` sem altera a version de v1 para v2.


```bash

docker build -t andremariadevops/api-rocket:v1 .

```


```bash

docker push andremariadevops/api-rocket:v1

```

![](image/Kubernetes/build-push-not-change-version.png)


![](image/Kubernetes/overview-image-vergion.png)

Agora será que no Kubernetes vamos conseguir fazer o deployment ?

Para testar vamos tentar acessar o novo end-point http://localhost:64730/example-k8s


![](image/Kubernetes/new-end-point-not-found.png)

Nesse caso para contornar esse problema vamos usar o `"imagePullPolicy:Always "`

Para isso vamos editar o arquuivo `deployment.yaml`

```yaml

apiVersion: apps/v1
kind: Deployment

metadata:
  name: api-rocket

spec:
  replicas: 2
  selector:
    matchLabels:
      api: api-rocket
  template:
    metadata:
      labels:
        api: api-rocket
    spec:
      containers:
      - name: api-rocket
        image: andremariadevops/api-rocket:v1
        imagePullPolicy: Always
        resources:
          requests:
            cpu: 100m
            memory: 64Mi
          limits:
            cpu: "200m"
            memory: "128Mi"
        ports:
        - containerPort: 3000

```

### Função do `"imagePullPolicy"`

O parâmetro `"imagePullPolicy"` é usado em ferramentas de orquestração, como Kubernetes, para controlar quando e como a imagem Docker deve ser baixada (ou puxada) do registro. Ele trabalha em conjunto com o versionamento para assegurar um comportamento previsível. 

### Valores Possíveis do `"imagePullPolicy"`:

1. **`Always`**:
   - Faz o download da imagem toda vez que o pod é iniciado.
   - Útil para tags não versionadas, como `latest`, garantindo que a versão mais recente esteja sempre em uso.
   - Pode aumentar o tempo de inicialização devido ao download constante da imagem.

2. **`IfNotPresent`**:
   - Apenas baixa a imagem se ela não estiver disponível no nó onde o pod está sendo executado.
   - Ideal para imagens versionadas, já que evita downloads desnecessários e aproveita as imagens já armazenadas localmente.

3. **`Never`**:
   - Nunca tenta baixar a imagem, assumindo que ela já está disponível localmente.
   - Geralmente usado em cenários controlados, como testes locais ou ambientes restritos sem acesso ao registro.

### Como o `"imagePullPolicy"` complementa o versionamento?

- Quando se usa **versionamento**, a política `IfNotPresent` é comumente utilizada para evitar downloads repetidos da mesma imagem, economizando largura de banda e acelerando o deployment.
- Para tags não versionadas, como `latest`, o uso de `Always` é recomendado para garantir que a versão mais recente da imagem seja usada. No entanto, essa prática é desencorajada em produção devido à falta de previsibilidade.


```bash

kubectl apply -f k8s/deployment.yaml -n ns-rocket

```

![](image/Kubernetes/download-image-kubernetes.png)


![](image/Kubernetes/new-end-point-ok.png)

## ✨ Entendendo Problemas da Tag Latest

Vimos que a versão é muito importante para o health da aplicação.

Agora vamos descobrir como fazer um rollback da aplicação nesse cenário de má prática de não utilizar o vercionamento da imagem.


```bash
kubectl rollout history deployment/api-rocket -n ns-rocket
```


### Explicação do comando `kubectl rollout history deployment/api-rocket`

O comando `kubectl rollout history deployment/api-rocket` é usado para exibir o histórico de revisões de um *Deployment* específico no Kubernetes. No caso do comando apresentado, ele está sendo aplicado ao *Deployment* chamado `api-rocket`.

### Funcionamento
- **`kubectl rollout`**: É o comando relacionado à administração de atualizações e mudanças de recursos do tipo *Deployment* no Kubernetes.
- **`history`**: Esta subcomando exibe o histórico de revisões do *Deployment*, incluindo informações sobre alterações realizadas em diferentes versões.
- **`deployment/api-rocket`**: Especifica o recurso (neste caso, um *Deployment* chamado `api-rocket`) para o qual o histórico deve ser consultado.

### Saída Esperada
A saída do comando geralmente exibe uma tabela com as seguintes colunas:
- **REVISION**: O número da revisão (começando em 1 para o primeiro estado registrado).
- **CHANGE-CAUSE**: Uma descrição sobre a causa da mudança, se fornecida no momento da aplicação do comando `kubectl apply` ou `kubectl rollout`.



### Quando usar
- Para identificar alterações no *Deployment* ao longo do tempo.
- Para verificar quem ou o que realizou mudanças.
- Para auxiliar em *rollbacks* ou diagnósticos de problemas relacionados a alterações.


![](image/Kubernetes/result-rollout-history.png)

### Adicionando o CHANGE-CAUSE
Para que o campo `CHANGE-CAUSE` seja preenchido, é necessário especificar a causa da mudança ao aplicar alterações, utilizando a flag `--record`, como no exemplo abaixo:

```bash
kubectl apply -f deployment.yaml --record
```


`"kubectl rollout undo"`: Reverte o Deployment para uma revisão anterior, útil quando algo deu errado após uma atualização.

```bash
kubectl rollout undo deployment/api-rocket --to-revision=1 -n ns-rocket
```

![](image/Kubernetes/rollback-version-kubernetes.png)

Agora vamos verificar e o end-point **`"/example-k8s"`** foi excluido das rotas da api.

![](image/Kubernetes/rote-error-example-k8s.png)

Como podemos ver a rota ainda existe. Mas por que?

Aqui o rollback faz o download da imagem ou reaproveita a mesma.
Como não alteramos a versão da imagem temos a perda o lastro, desta forma o rollback não foi realizado corretamente.


## ✨ Criando Nova Tag e Controlando Rollback da Aplicação

Agora que entendemos que não é uma boa prática sobre-escrever tags, vamos fazer algumas alterações.

Nesse caso para contornar esse problema vamos usar o `"imagePullPolicy:IfNotPresent "`

Para isso vamos editar o arquuivo `deployment.yaml`

```yaml

apiVersion: apps/v1
kind: Deployment

metadata:
  name: api-rocket

spec:
  replicas: 2
  selector:
    matchLabels:
      api: api-rocket
  template:
    metadata:
      labels:
        api: api-rocket
    spec:
      containers:
      - name: api-rocket
        image: andremariadevops/api-rocket:v2
        imagePullPolicy: IfNotPresent
        resources:
          requests:
            cpu: 100m
            memory: 64Mi
          limits:
            cpu: "200m"
            memory: "128Mi"
        ports:
        - containerPort: 3000

```



```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Rocketset Api!';
  }

  getExample(): string {
    
    return `Estou rodando no K8s! ${ Date.UTC }`;
  }
}
```

```bash

docker build -t andremariadevops/api-rocket:v2 .

```


```bash

docker push andremariadevops/api-rocket:v2

```

![](image/Kubernetes/deploment-v2-api.png)


![](image/Kubernetes/docker-hub-new-version.png)

```bash

 kubectl apply -f k8s -n ns-rocket

```

![](image/Kubernetes/new-end-point-change-result.png)


```
PS C:\Repo\rocketseat.ci.api> kubectl rollout history deployment/api-rocket -n ns-rocket
deployment.apps/api-rocket 
REVISION  CHANGE-CAUSE
2         <none>
3         <none>
4         <none>

PS C:\Repo\rocketseat.ci.api> 

```

**`"!Aqui todos os comando executados no terminal foram executados de forma imperativa!"`**

## ✨ Trabalhando Com Estratégias De Deploy

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: api-rocket

spec:
  replicas: 2
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      api: api-rocket
  template:
    metadata:
      labels:
        api: api-rocket
    spec:
      containers:
      - name: api-rocket
        image: andremariadevops/api-rocket:v2
        imagePullPolicy: IfNotPresent
        resources:
          requests:
            cpu: 100m
            memory: 64Mi
          limits:
            cpu: "200m"
            memory: "128Mi"
        ports:
        - containerPort: 3000
```

## ✨ Entendendo o Recreate

## ✨ Explorando Variável de Ambiente na Aplicação

## ✨ Entendendo sobre o ConfigMap

## ✨ Explorando o objeto Secret

## ✨ Melhorando Gerenciamento de Envs