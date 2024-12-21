#### `apiVersion: v1`
- Define a versão da API do Kubernetes utilizada para este recurso.
- `v1` é a versão estável para recursos básicos como `Pod`.
#### `kind: Pod`
- Especifica o tipo de recurso que será criado.
- Neste caso, estamos criando um **Pod**, que é a menor unidade executável no Kubernetes.
#### `metadata:`
- Define informações adicionais sobre o recurso.
#### `name: nginx`
- Atribui o nome **nginx** ao Pod.
- Este nome deve ser único dentro do namespace onde o Pod está sendo criado.
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
#### Resumo
Este script YAML configura um Pod com:
- Um container baseado na imagem **nginx:stable-alpine3.20-perl**.
- Exposição da porta 80.
- Garantia de uso mínimo de **0,1 vCPU** e **64 MiB** de memória.
- Limite máximo de **0,2 vCPU** e **128 MiB** de memória.
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
#### 1. **ReplicaSet Monitora o Estado**
O ReplicaSet verifica constantemente o número de pods em execução, baseando-se no `selector.matchLabels`. 
#### 2. **Ação Após a Deleção**
- Ao deletar um pod gerenciado pelo ReplicaSet, ele detecta a ausência desse pod e recria um novo imediatamente.
- O novo pod será baseado no modelo definido na seção `template` do ReplicaSet.
#### 3. **Imutabilidade dos Pods**
- Os pods criados pelo ReplicaSet são independentes e imutáveis. Alterar diretamente um pod (como modificar sua configuração) não afeta o modelo do ReplicaSet.
- Se um pod for modificado manualmente, o ReplicaSet substituirá esse pod por outro que siga o modelo original.
#### 4. **Impacto no Cluster**
- Caso todos os pods gerenciados sejam deletados, o ReplicaSet recriará os pods até atingir o número especificado em `replicas`.
- Se os recursos do cluster (CPU, memória, etc.) forem insuficientes, o ReplicaSet continuará tentando recriar os pods até que os recursos estejam disponíveis.
#### Exceção: Deleção do Próprio ReplicaSet
- Se o **ReplicaSet** for deletado, todos os pods gerenciados por ele também serão removidos, já que dependem do ReplicaSet para existir.
#### Caso Prático
Você pode testar a recriação automática dos pods com os seguintes comandos:
#### Usando o ReplicaSet
```yaml
apiVersion: apps/v1
kind: ReplicaSet
#### **`"apiVersion: apps/v1"`** 
Define a versão da API utilizada para o recurso. Nesse caso, o ReplicaSet usa a API `apps/v1`, que é estável para gerenciar aplicativos no Kubernetes.
#### **`"kind: ReplicaSet"`** 
Especifica o tipo de recurso que está sendo criado. Aqui, o recurso é um **ReplicaSet**.
#### **`"metadata:"`** 
Contém informações básicas de identificação do objeto.
#### **`"spec:"`** 
Define a especificação desejada para o ReplicaSet, como o número de réplicas e a configuração dos pods.
#### **`"replicas: 5"`** 
Especifica o número desejado de réplicas do pod. O ReplicaSet garante que exatamente 5 pods estejam sempre em execução.
#### **`"selector:"`** 
Define como o ReplicaSet identifica os pods que ele deve gerenciar.
#### **`"template:"`** 
Define o modelo (template) para os pods que o ReplicaSet irá criar. Essa seção é usada para configurar os pods.
#### **`"containers:"`** 
Lista os containers que devem ser executados no pod.
#### **`"resources:"`** 
Define as solicitações e limites de recursos para o container.
#### Resumo
Este manifesto cria um **`"ReplicaSet"`** chamado `nginx`, que mantém 5 réplicas de um pod executando um container NGINX baseado na imagem `nginx:stable-alpine3.20-perl`. 
Os pods são configurados para escutar na porta 80 e possuem restrições de CPU e memória definidas para otimizar o uso de recursos do cluster.
#### Considerações Finais
Embora o ReplicaSet seja a base para recursos como o Deployment, seu uso direto é recomendado apenas para casos muito específicos, como:
#### Resumo do Fluxo
1. **Criar o Service**:
   - O Kubernetes identifica que é um recurso do tipo `Service`.
2. **Filtrar Pods**:
   - Usa o seletor `app: nginx` para encontrar os pods associados.
3. **Redirecionar o Tráfego**:
   - O tráfego enviado para o `Service` na porta `80` é redirecionado para a porta `80` nos containers selecionados.
### Explicação detalhada do comando
**Comando:**  
`kubectl port-forward svc/nginx-svc -n first-app 8080:80`
#### 1. **Contexto**  
Este comando é usado para criar um túnel local que encaminha o tráfego de uma porta no computador do usuário para uma porta em um serviço (Service) no cluster Kubernetes.
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
#### 3. **Uso prático**  
Após executar este comando, você pode acessar o serviço `nginx-svc` diretamente no navegador ou com ferramentas como `curl` via:  
#### 4. **Cenários de uso comuns**
- Testar um serviço no cluster sem configurá-lo como um recurso publicamente acessível.  
- Depuração ou desenvolvimento local, acessando serviços internos do Kubernetes de forma segura.  
#### 5. **Notas importantes**
- O comando precisa de acesso ao cluster Kubernetes configurado no `kubectl`.  
- Certifique-se de que a porta local (8080) não esteja em uso por outro processo antes de executar o comando.  
- Se o Service estiver configurado corretamente no cluster, o tráfego será redirecionado para os pods associados a ele.
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
### Adicionando o CHANGE-CAUSE
Para que o campo `CHANGE-CAUSE` seja preenchido, é necessário especificar a causa da mudança ao aplicar alterações, utilizando a flag `--record`, como no exemplo abaixo:
#### 1. Troca Gradual de Pods
- O Kubernetes cria novos pods com a versão atualizada da aplicação.
- Em seguida, remove os pods antigos, um por vez, ou conforme a configuração.
#### 2. Configurações Principais
No arquivo YAML do Deployment ou StatefulSet, você pode configurar os seguintes parâmetros no campo `strategy`:
#### 3. Interrupção de Atualizações
- Se algo der errado, é possível interromper (pause) ou reverter a atualização (rollback) para evitar downtime ou problemas maiores.
#### 4. Comandos úteis
- **Pausar** uma atualização:
  ```bash
  kubectl rollout pause deployment/my-deployment
  ```
- **Retomar** a atualização:
  ```bash
  kubectl rollout resume deployment/my-deployment
  ```
- **Verificar o status** da atualização:
  ```bash
  kubectl rollout status deployment/my-deployment
  ```
- **Reverter** para uma versão anterior:
  ```bash
  kubectl rollout undo deployment/my-deployment
  ```
#### 5. Vantagens do RollingUpdate
- **Sem downtime**: Garante disponibilidade durante o processo.
- **Controlável**: Permite configurar tolerâncias para indisponibilidade e escalar gradualmente.
#### 6. Limitações
- Pode ser mais lento que uma substituição completa.
- Não é ideal para cenários em que é necessária consistência absoluta entre todos os pods (nesse caso, pode-se usar a estratégia **Recreate**).
#### O que é a estratégia `Recreate`?
- **Comportamento**: Na estratégia `Recreate`, todos os pods do conjunto antigo são **primeiro terminados** antes que os novos sejam criados. Isso significa que haverá um período de indisponibilidade enquanto a transição ocorre.
- **Configuração**: Essa estratégia é útil para aplicações que não suportam múltiplas versões em execução ao mesmo tempo ou quando o estado compartilhado entre as versões pode causar conflitos.
#### Quando usar `Recreate`?
- **Aplicações com estado**: Quando sua aplicação mantém um estado que pode ser corrompido se múltiplas instâncias (ou versões diferentes) forem executadas simultaneamente.
- **Conexões exclusivas**: Se o aplicativo usa conexões de longa duração, filas, ou qualquer recurso exclusivo que não permita concorrência entre múltiplas versões.
- **Compatibilidade restrita**: Quando não há compatibilidade retroativa ou compatibilidade entre versões (por exemplo, uma aplicação com mudanças radicais no banco de dados ou API).
#### Vantagens do `Recreate`
1. **Simplicidade**: A estratégia elimina a necessidade de gerenciar múltiplas versões simultaneamente.
2. **Evita conflitos**: Ideal para cenários em que as versões do aplicativo podem interferir uma na outra.
3. **Previsibilidade**: Como todos os pods antigos são encerrados antes de iniciar os novos, é mais fácil garantir consistência.
#### Desvantagens do `Recreate`
1. **Indisponibilidade**: Haverá um período de downtime entre o término dos pods antigos e o início dos novos.
2. **Impacto em usuários**: Pode não ser adequado para aplicações críticas onde a disponibilidade contínua é essencial.
#### Dicas ao usar `Recreate`
- **Planeje janelas de manutenção**: Use a estratégia durante períodos de baixa demanda para minimizar o impacto nos usuários.
- **Monitore os recursos**: Certifique-se de que os novos pods podem ser inicializados rapidamente para reduzir o downtime.
- **Use readiness probes**: Isso ajuda a garantir que os novos pods estejam totalmente funcionais antes de considerar a atualização como concluída.
#### 1. Instalar Dependências
Primeiro, instale o pacote necessário para trabalhar com variáveis de ambiente:
#### 2. Criar um Arquivo `.env`
Crie um arquivo `.env` na raiz do seu projeto e adicione suas variáveis:
#### 3. Configurar o Módulo de Configuração no AppModule
No arquivo `app.module.ts`, importe e configure o módulo de configuração:
#### 4. Consumir Variáveis no Código
Use o serviço `AppService` para acessar as variáveis de ambiente. Você pode injetar o `ConfigService` em qualquer lugar.
#### Propósito do Script
A definição de um **ConfigMap** no Kubernetes. Um **ConfigMap** é usado para armazenar pares de chave-valor que podem ser utilizados pelas aplicações em contêiners, 
permitindo a configuração de aplicações sem necessidade de alterar suas imagens.
#### 1. `apiVersion: v1`
- **Significado:** Define a versão da API do Kubernetes que será usada para criar este objeto.
- **Detalhes:** A versão `v1` é a versão estável e comumente usada para objetos como ConfigMaps.
#### 2. `kind: ConfigMap`
- **Significado:** Especifica o tipo de recurso que está sendo definido.
- **Detalhes:** Neste caso, o recurso é um ConfigMap, que será usado para armazenar dados de configuração em forma de texto simples.
#### 3. `metadata:`
- **Significado:** Contém metadados sobre o recurso.
- **Detalhes:** Esta seção inclui informações como o nome do ConfigMap e, opcionalmente, labels e anotações para identificação e organização.
##### 3.1. `name: api-rocket`
- **Significado:** Define o nome do ConfigMap.
- **Detalhes:** O nome é `api-rocket`, que será usado para referenciar este ConfigMap em outros recursos do Kubernetes.
#### 4. `data:`
- **Significado:** Contém os dados de configuração em formato de pares de chave-valor.
- **Detalhes:** Os dados aqui definidos podem ser usados por aplicações para configurar seu funcionamento.
##### 4.1. `app-name: rocketseat-app`
- **Significado:** Define um par chave-valor onde:
  - `app-name` é a chave.
  - `rocketseat-app` é o valor associado à chave.
- **Detalhes:** Este valor pode ser utilizado por uma aplicação em execução para, por exemplo, identificar o nome do aplicativo.
### Exemplo de Uso
- Este ConfigMap pode ser montado como um arquivo ou passado como variável de ambiente para os contêiners que fazem parte de um **Pod** no Kubernetes.
### Benefícios do Uso do ConfigMap
- **Centralização de Configuração:** Facilita a alteração de parâmetros sem a necessidade de reconstruir as imagens dos contêineres.
- **Flexibilidade:** Permite que a mesma imagem seja usada em diferentes ambientes com configurações distintas.
- **Manutenção:** Facilita a gestão de configurações em ambientes dinâmicos e escaláveis.
#### 1. Manutenção Simplificada
- **Comentado**:
  - Variáveis de ambiente específicas (`APP` e `API_KEY`) são configuradas diretamente no `Deployment`. Caso outras variáveis precisem ser adicionadas ou modificadas, o arquivo do `Deployment` precisaria ser alterado.
#### 2. Segregação de Responsabilidades
- **Comentado**:
  - Todas as definições de variáveis (tanto sensíveis quanto não sensíveis) estão no mesmo local.
#### 3. Escalabilidade e Reutilização
- **Comentado**:
  - Cada variável de ambiente precisava ser explicitamente mapeada, tornando o arquivo de configuração menos reutilizável em diferentes cenários.
#### 4. Redução de Erros
- **Comentado**:
  - O uso de `configMapKeyRef` e `secretKeyRef` exige especificar cada chave e pode levar a erros caso o nome ou a chave estejam incorretos.
##### **ConfigMap**
- **Comentado**:
  - Definição direta da chave `app-name`, que é referenciada no `Deployment`.
- **Atual**:
  - Alterado para usar diretamente a variável `APP`, facilitando o carregamento pelo `envFrom` e mantendo consistência com o `.env`.
##### **Secret**
- **Comentado**:
  - Chave nomeada como `api-key`, usada diretamente no `Deployment`.
- **Atual**:
  - Alterada para `API_KEY`, mantendo consistência com o `.env` e simplificando o carregamento via `envFrom`.
#### Vantagens Adicionais
1. **Conformidade com Boas Práticas**: A separação entre ConfigMap e Secret é alinhada às práticas recomendadas do Kubernetes.
2. **Integração com Ferramentas DevOps**: O uso de `.env` como referência facilita a integração com pipelines de CI/CD e evita discrepâncias entre o ambiente local e o Kubernetes.
3. **Escalabilidade**: O novo formato suporta com facilidade a adição de mais variáveis, sem necessidade de alterações estruturais no `Deployment`.
#### **Cabeçalho**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
```
#### 1. `kubectl run`
- **`kubectl`**: Ferramenta CLI para interagir com clusters Kubernetes.
- **`run`**: Cria e executa um pod no cluster Kubernetes.
#### 2. Nome do pod: `fortio`
- Define o nome do pod temporário como **`fortio`**.
#### 3. `-it`
- **`-i`**: Habilita entrada interativa para o pod.
- **`-t`**: Ativa o modo de terminal, conectando seu terminal local ao processo do pod.
#### 4. `-n ns-rocket`
- Especifica o **namespace** Kubernetes onde o pod será criado.
- **`ns-rocket`**: Nome do namespace.
#### 5. `--rm`
- Remove automaticamente o pod após sua execução. Isso evita acumular pods temporários.
#### 6. `--image=fortio/fortio`
- Define a **imagem Docker** a ser usada.
- **`fortio/fortio`**: Imagem da ferramenta Fortio, usada para testes de desempenho e carga.
#### 7. `-- load`
- Ativa o modo **teste de carga** no Fortio.
#### 8. `-qps 6000`
- Define o número de **requisições por segundo (QPS)**.
- **`6000`**: Envia **6.000 requisições por segundo**.
#### 9. `-t 120s`
- Especifica a **duração do teste**.
- **`120s`**: O teste será executado por **120 segundos (2 minutos)**.
#### 10. `-c 50`
- Define o número de **conexões simultâneas (threads)**.
- **`50`**: Usa **50 conexões simultâneas** durante o teste.
#### 11. `"http://api-rocket-svc/example-k8s"`
- Define o **endpoint** a ser testado.
- **`http://api-rocket-svc/example-k8s`**: URL do serviço que será submetido ao teste de carga.
- Como está dentro do Cluster é possível enxergar o serviço.
---
#### `stabilizationWindowSeconds: 5`
- Especifica uma janela de estabilização de **5 segundos**.
- Isso significa que o HPA esperará **5 segundos** antes de aumentar o número de réplicas, mesmo que as métricas indiquem a necessidade de escalonamento.
- Esse intervalo reduz oscilações rápidas no número de réplicas, garantindo que o aumento seja consistente.
#### `policies`
Define as políticas que limitam como o escalonamento para cima ocorrerá. Aqui, há apenas uma política ativa:
#### Janela de Estabilização (`stabilizationWindowSeconds`)
- Serve para prevenir oscilações rápidas no número de réplicas causadas por variações momentâneas na carga.
#### Política Baseada em Réplicas Absolutas (`type: Pods`)
- Permite um aumento controlado no número de réplicas.
- Por exemplo, se há **4 réplicas** atualmente e a carga exige mais réplicas, o HPA só adicionará **até 2 réplicas** em intervalos de **5 segundos**, mesmo que mais réplicas sejam necessárias.
#### **Controle Fino do Escalonamento**
- Evita um aumento excessivo e rápido no número de réplicas, o que pode levar ao uso ineficiente de recursos.
- Permite um ajuste gradual e controlado.
#### **Estabilidade**
- Reduz oscilações no número de réplicas, mesmo em cenários de carga variável.
#### **Respostas Rápidas**
- A janela de estabilização curta (**5 segundos**) permite respostas quase imediatas em situações de alta demanda.
### **Como funciona?**
- O Kubernetes monitora os containers continuamente usando as probes.
- Se uma **liveness probe** falhar, o container será reiniciado automaticamente.
- Se um nó do cluster falhar, os pods que estavam nesse nó são redistribuídos para outros nós disponíveis.
- Se uma **readiness probe** falhar, o tráfego não é roteado para aquele pod até que ele volte a estar saudável.
### **Benefícios**:
- Redução de downtime (tempo de inatividade).
- Menor necessidade de intervenção manual.
- Maior confiabilidade e estabilidade da aplicação.
#### startupProbe
- **`startupProbe`** verifica a saúde do contêiner **durante o processo de inicialização**.
- Enquanto a sonda não for bem-sucedida, o Kubernetes considera que o contêiner **ainda não está pronto** para aceitar tráfego ou executar outras operações.
#### httpGet
- Especifica que a sonda fará uma requisição HTTP para verificar o estado da aplicação.
  - **path: /healthz**  
    Define o endpoint no contêiner que será verificado para determinar a saúde do contêiner.
  - **port: 3000**  
    Porta do contêiner onde será feita a requisição HTTP.
#### failureThreshold
- **3**  
  Número de falhas consecutivas permitidas antes que o Kubernetes considere que o contêiner **não conseguiu inicializar**.  
  Após 3 falhas consecutivas, o contêiner será reiniciado.
#### successThreshold
- **1**  
  Número de verificações bem-sucedidas consecutivas necessárias para o Kubernetes considerar o contêiner **pronto**.  
  Aqui, basta **1 verificação bem-sucedida**.
#### timeoutSeconds
- **1**  
  Tempo máximo (em segundos) que o Kubernetes espera por uma resposta do endpoint antes de considerar a tentativa como falha.
#### periodSeconds
- **10**  
  Intervalo de tempo (em segundos) entre cada execução da sonda.
#### Funcionamento Geral
1. O Kubernetes verifica periodicamente o endpoint HTTP `/healthz` na porta 3000 assim que o contêiner começa a iniciar.
2. Caso o endpoint responda com sucesso (ex.: HTTP 200), o contêiner será considerado como inicializado.
3. Se o contêiner falhar 3 vezes consecutivas (por timeout ou erro no endpoint), ele será reiniciado.
4. A sonda é executada a cada 10 segundos, e o Kubernetes espera no máximo 1 segundo pela resposta em cada verificação.
#### NestFactory
- É uma classe fornecida pelo NestJS que facilita a criação de instâncias de aplicativos.
- `NestFactory.create()` é o método usado para criar a aplicação principal.
#### AppModule
- É o módulo raiz da aplicação. Ele geralmente serve como ponto de entrada, onde outros módulos, controladores e provedores são importados e configurados.
- Esse arquivo `app.module.ts` normalmente define os componentes principais que sua aplicação usará.
#### Definição da função bootstrap
- A função bootstrap é uma função assíncrona que serve como o ponto de entrada para a aplicação.
- O NestJS é baseado em programação assíncrona devido à natureza do Node.js e seu loop de eventos.
#### AppModule
- O NestJS usa o conceito de módulos, e o `AppModule` é o módulo inicial, onde você configura rotas, controladores, serviços, middlewares, etc.
#### Arquitetura modular
- O código segue o padrão arquitetural orientado a módulos, permitindo uma organização mais limpa e escalável.
## Volumes e StorageClass
No Kubernetes, volumes são recursos utilizados para armazenar dados persistentes que podem ser acessados por contêineres, 
mesmo que estes sejam reiniciados ou recriados. 
## PersistentVolume (PV)
Um `PersistentVolume` é um recurso no Kubernetes que representa uma unidade de armazenamento configurada de forma independente 
do ciclo de vida dos pods. 
## PersistentVolumeClaim (PVC)
Um `PersistentVolumeClaim` é uma solicitação feita por um pod para utilizar um `PersistentVolume`. 
## Criando o StorageClass
O `StorageClass` é criado por meio de um manifesto YAML. 
## Reservando o espaço com o PV
O `PersistentVolume` é criado manualmente ou automaticamente com base em um `StorageClass`. 
## Criando o PVC
O `PersistentVolumeClaim` é criado para solicitar um volume com base nas especificações desejadas. 
