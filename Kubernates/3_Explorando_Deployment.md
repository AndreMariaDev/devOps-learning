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

#### RollingUpdate

O **RollingUpdate** no Kubernetes é uma estratégia de atualização utilizada para atualizar os pods de forma controlada e sem causar downtime no serviço. Essa estratégia é usada no **Deployment** e no **StatefulSet**, permitindo substituir gradualmente os pods antigos por novos, garantindo que sempre haja um número mínimo de pods disponíveis durante o processo.

#### Como funciona o RollingUpdate?

#### 1. Troca Gradual de Pods
- O Kubernetes cria novos pods com a versão atualizada da aplicação.
- Em seguida, remove os pods antigos, um por vez, ou conforme a configuração.

#### 2. Configurações Principais
No arquivo YAML do Deployment ou StatefulSet, você pode configurar os seguintes parâmetros no campo `strategy`:

- **`maxUnavailable`**: Define o número máximo de pods que podem estar indisponíveis durante a atualização.
- **`maxSurge`**: Especifica quantos pods adicionais podem ser criados além do número desejado de réplicas.

#### Exemplo:

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

Se precisar de um exemplo mais detalhado ou ajustes para um caso específico, me avise!


## ✨ Entendendo o Recreate

#### Estratégia `Recreate` no Kubernetes

A estratégia **`Recreate`** no Kubernetes é uma das duas principais estratégias de atualização disponíveis (a outra é **`RollingUpdate`**). Essa estratégia é utilizada em controladores como o **`Deployment`** e o **`StatefulSet`**, definindo como os pods devem ser gerenciados durante uma atualização.

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

A estratégia `Recreate` é um bom ajuste para cenários específicos, mas, para a maioria das aplicações modernas, a **estratégia `RollingUpdate`** é mais recomendada por evitar downtime.

## ✨ Explorando Variável de Ambiente na Aplicação

#### Configurar e Consumir Informações de um Arquivo `.env` no NestJS

No NestJS, você pode configurar e consumir informações de um arquivo `.env` usando o pacote `@nestjs/config`. Abaixo está um guia passo a passo:

---

#### 1. Instalar Dependências
Primeiro, instale o pacote necessário para trabalhar com variáveis de ambiente:

```bash
npm install @nestjs/config dotenv
```

---

#### 2. Criar um Arquivo `.env`
Crie um arquivo `.env` na raiz do seu projeto e adicione suas variáveis:


```env
APP=rocketseat-app
```

#### 3. Configurar o Módulo de Configuração no AppModule
No arquivo `app.module.ts`, importe e configure o módulo de configuração:

```typescript
import { Module } from '@nestjs/common';
import { ConfigModule } from '@nestjs/config';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true, // Torna o módulo global, não precisa importar em outros módulos
      envFilePath: '.env', // Caminho do arquivo .env (opcional, padrão é .env)
    }),
  ],
})
export class AppModule {}
```
#### 4. Consumir Variáveis no Código
Use o serviço `AppService` para acessar as variáveis de ambiente. Você pode injetar o `ConfigService` em qualquer lugar.

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class AppService {
  getHello(): string {
    return 'Rocketset Api!';
  }

  getExample(): string {
    
    return `Estou rodando no K8s! ${ Date.UTC }: ${process.env.APP}`;
  }
}
```

Agora, seu aplicativo NestJS está configurado para consumir e validar variáveis de ambiente do arquivo `.env`! 🚀

## ✨ Entendendo sobre o ConfigMap

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: api-rocket

spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 20%
      maxSurge: 10%
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

```bash

kubectl apply -f k8s/deployment.yaml -n ns-rocket

```

```bash

docker build -t andremariadevops/api-rocket:v3 .

```


```bash

docker push andremariadevops/api-rocket:v3

```

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: api-rocket

spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 20%
      maxSurge: 10%
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
        image: andremariadevops/api-rocket:v3
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

```bash

kubectl apply -f k8s/deployment.yaml -n ns-rocket

```

![](image/Kubernetes/undefined-variable.png)


Como podemos ver o valor entre a APP do env e a aplicação estão indefinidos. Para resolvermos esse problema vamos criar um novo arquivo 
`configmap.yaml`.


```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: api-rocket
data:
  app-name: rocketseat-app

```


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



```bash

kubectl apply -f k8s/configmap.yaml -n ns-rocket

```

![](image/Kubernetes/confirm-variable-configmap.png)

```bash

kubectl apply -f k8s/deployment.yaml -n ns-rocket

```


![](image/Kubernetes/ok-variable.png)

## ✨ Explorando o objeto Secret


Vamos criar um novo arquivo 
`secret.yaml`.

```yaml
apiVersion: v1
kind: Secret

metadata:
  name:  api-rocket-secrets
type: Opaque
data:
  api-key: cm9ja2V0c2VhdC1hcHA=

```

`app-key: cm9ja2V0c2VhdC1hcHA=` o valor da `app-key` deve ser em base 64 


```bash

kubectl apply -f k8s/secret.yaml -n ns-rocket

```

```bash

docker build -t andremariadevops/api-rocket:v4 .

```


```bash

docker push andremariadevops/api-rocket:v4

```

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: api-rocket

spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 20%
      maxSurge: 10%
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
        image: andremariadevops/api-rocket:v4
        imagePullPolicy: IfNotPresent
        env:
          - name:  APP
            valueFrom:
              configMapKeyRef:
                name:  api-rocket
                key:  app-name
          - name:  API_KEY
            valueFrom:
              secretKeyRef:
                name:  api-rocket-secrets
                key:  api-key
        resources:
          requests:
            cpu: 100m
            memory: 64Mi
          limits:
            cpu: "200m"
            memory: "128Mi"
        ports:
        - containerPort: 3000
        - containerPort: 3000
```
![](image/Kubernetes/secret-kube.png)


```bash

kubectl apply -f k8s/deployment.yaml -n ns-rocket

```

## ✨ Melhorando Gerenciamento de Envs

![](image/Kubernetes/change-implementation-secret-variable.png)

```yaml

apiVersion: apps/v1
kind: Deployment

metadata:
  name: api-rocket

spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 20%
      maxSurge: 10%
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
        image: andremariadevops/api-rocket:v4
        imagePullPolicy: IfNotPresent
        envFrom:
          - configMapRef:
              name: api-rocket
          - secretRef:
              name: api-rocket-secrets
        # env:
        #   - name:  APP
        #     valueFrom:
        #       configMapKeyRef:
        #         name:  api-rocket
        #         key:  app-name
        #   - name:  API_KEY
        #     valueFrom:
        #       secretKeyRef:
        #         name:  api-rocket-secrets
        #         key:  api-key
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

```yaml
apiVersion: v1
kind: ConfigMap

metadata:
  name: api-rocket

data:
  APP: rocketseat-app
  # app-name: rocketseat-app

```

```yaml
apiVersion: v1
kind: Secret

metadata:
  name:  api-rocket-secrets
type: Opaque
data:
  API_KEY: cm9ja2V0c2VhdC1hcHA=
  # api-key: cm9ja2V0c2VhdC1hcHA=

```

#### Explicação das Mudanças nos Trechos Comentados do Deployment

No **Deployment** do Kubernetes, o trecho comentado utilizava variáveis de ambiente específicas referenciadas diretamente nos campos `env` (`APP` e `API_KEY`). No trecho atual, optou-se por utilizar os recursos `envFrom` para importar variáveis de ambiente de um **ConfigMap** e de um **Secret**. Essa alteração traz as seguintes vantagens:

---

#### 1. Manutenção Simplificada
- **Comentado**:
  - Variáveis de ambiente específicas (`APP` e `API_KEY`) são configuradas diretamente no `Deployment`. Caso outras variáveis precisem ser adicionadas ou modificadas, o arquivo do `Deployment` precisaria ser alterado.

- **Atual**:
  - O uso de `envFrom` permite que todas as variáveis contidas no `ConfigMap` e no `Secret` sejam automaticamente carregadas, evitando alterações repetidas no `Deployment`.

---

#### 2. Segregação de Responsabilidades
- **Comentado**:
  - Todas as definições de variáveis (tanto sensíveis quanto não sensíveis) estão no mesmo local.

- **Atual**:
  - Separar as variáveis sensíveis (`API_KEY`, no `Secret`) e não sensíveis (`APP`, no `ConfigMap`) melhora a organização e facilita o controle. **ConfigMaps** são usados para dados não confidenciais, enquanto **Secrets** são criptografados.

---

#### 3. Escalabilidade e Reutilização
- **Comentado**:
  - Cada variável de ambiente precisava ser explicitamente mapeada, tornando o arquivo de configuração menos reutilizável em diferentes cenários.

- **Atual**:
  - `ConfigMap` e `Secret` podem ser reutilizados por outros **Deployments** ou serviços que precisem das mesmas variáveis. Isso elimina redundâncias e simplifica a gestão de configurações.

---

#### 4. Redução de Erros
- **Comentado**:
  - O uso de `configMapKeyRef` e `secretKeyRef` exige especificar cada chave e pode levar a erros caso o nome ou a chave estejam incorretos.

- **Atual**:
  - `envFrom` reduz a possibilidade de erros, pois todas as chaves válidas no `ConfigMap` ou no `Secret` são automaticamente carregadas.

---

#### Alterações no **ConfigMap** e **Secret**

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

---

#### Vantagens Adicionais
1. **Conformidade com Boas Práticas**: A separação entre ConfigMap e Secret é alinhada às práticas recomendadas do Kubernetes.
2. **Integração com Ferramentas DevOps**: O uso de `.env` como referência facilita a integração com pipelines de CI/CD e evita discrepâncias entre o ambiente local e o Kubernetes.
3. **Escalabilidade**: O novo formato suporta com facilidade a adição de mais variáveis, sem necessidade de alterações estruturais no `Deployment`.



```bash

kubectl apply -f k8s/secret.yaml -n ns-rocket

```

```bash

kubectl apply -f k8s/configmap.yaml -n ns-rocket

```

```bash

kubectl apply -f k8s/deployment.yaml -n ns-rocket

```


