## Contexto inicial e problema

### Cenário

- `"Temos Uma aplicação executando em container e essa execução em container traz algumas preocupações (Problemas)."`
- `"Temos várias aplicações executando em container e essa execução em container traz algumas preocupações (Problemas)."`

### Problemas

- `"E se um container falhar na execução?"`
- `"E se precisarmos de vários containers executando a mesma aplicação?"`
- `"E se precisarmos de um fluxo elástico?"`
- `"E se forem várias aplicações com vários containers?"`
- `"E o controle de uso de recursos de container?"`

## Esboço do problema

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

## Principais componentes

![](image/Kubernetes/arq-kubernetes.png)

#### Arquitetura do Kubernetes

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


## Como executar um Cluster Kubernetes

## Configurando Ambiente

## Configurando e criando nosso primeiro Cluster

## Configurando Cluster com multiplos nós


- `""`