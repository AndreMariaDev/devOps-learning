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

