# Probes e Self Healing

## O que são Probes e Self Healing

No Kubernetes, **Probes** e o conceito de **Self-Healing** estão relacionados ao gerenciamento de aplicações e à resiliência do sistema. Veja a explicação detalhada:

---

### **Probes**

Probes são mecanismos usados para monitorar a saúde dos containers em um cluster. Elas permitem ao Kubernetes determinar se um container está funcionando corretamente e tomar as ações necessárias em caso de falhas. Existem três tipos principais de probes:

### **Tipos de Probes**

1. **Liveness Probe**:
   - Verifica se o container está "vivo" ou funcionando corretamente.
   - Se falhar, o Kubernetes reinicia o container, assumindo que ele está travado ou em um estado irreparável.
   - **Exemplo**: Verificar se um processo essencial está em execução.

2. **Readiness Probe**:
   - Verifica se o container está "pronto" para receber tráfego.
   - Se falhar, o container é retirado do **Service** até que passe novamente.
   - **Exemplo**: Certificar-se de que a aplicação carregou completamente e está pronta para responder a solicitações.

3. **Startup Probe**:
   - Verifica se o container foi iniciado corretamente. É útil para aplicações que demoram muito para inicializar.
   - Evita que o Kubernetes reinicie prematuramente containers que demoram para ficar prontos.

### **Métodos de Configuração**

As probes podem ser configuradas de diferentes formas:
- **HTTP**: Faz uma solicitação HTTP para verificar o status (ex.: código 200 é considerado saudável).
- **TCP**: Verifica se consegue abrir uma conexão TCP com o container.
- **Command**: Executa um comando no container e verifica o código de saída (0 é considerado sucesso).

---

### **Self-Healing (Auto-Cura)**

O **Self-Healing** é um dos princípios fundamentais do Kubernetes, garantindo a alta disponibilidade e a resiliência das aplicações. Ele utiliza probes e outros mecanismos para detectar falhas e corrigir problemas automaticamente.

### **Como funciona?**
- O Kubernetes monitora os containers continuamente usando as probes.
- Se uma **liveness probe** falhar, o container será reiniciado automaticamente.
- Se um nó do cluster falhar, os pods que estavam nesse nó são redistribuídos para outros nós disponíveis.
- Se uma **readiness probe** falhar, o tráfego não é roteado para aquele pod até que ele volte a estar saudável.

### **Benefícios**:
- Redução de downtime (tempo de inatividade).
- Menor necessidade de intervenção manual.
- Maior confiabilidade e estabilidade da aplicação.

## Configurando Rotas na Aplicação

Na aplicação Nest.js vamos criar um novo controller. Para isso vamos criar uma nova pasta nomeada `health` contendo um controller e um service.

![](image/Kubernetes/create-new-componente.png)



No arquivo `health.service.ts` temos :

```typescript

import { Injectable } from '@nestjs/common';

@Injectable()
export class HealthService {

  checkHealth(): string {
    return 'Check Health Rocketset Api OK!';
  }

  checkReady(): string {
    return 'Check Ready Rocketset Api OK!';
  }
}


```

No arquivo `health.controller.ts` temos :

```typescript

import { Controller, Get } from '@nestjs/common';
import { HealthService } from './health.service';

@Controller()
export class HealthController {
  constructor(private readonly healthService: HealthService) {}

  @Get('/healthz')
  healthz(): string {
    return this.healthService.checkHealth();
  }

  @Get('/readyz')
  readyz(): string {
    return this.healthService.checkReady();
  }
}


```

Agora é necessário declarar os novos arquivos no `app.modules.ts`.

```typescript

import { Module } from '@nestjs/common';
import { AppController } from './app.controller';
import { AppService } from './app.service';
import { ConfigModule } from '@nestjs/config';
import { HealthController } from './health/health.controller';
import { HealthService } from './health/health.service';

@Module({
  imports: [
    ConfigModule.forRoot({
      isGlobal: true, // Torna o módulo global, não precisa importar em outros módulos
      envFilePath: '.env', // Caminho do arquivo .env (opcional, padrão é .env)
    }),
  ],
  controllers: [AppController, HealthController],
  providers: [AppService, HealthService],
})
export class AppModule {}


```

Agora vamos excutar os seguintes comandos:

```bash

docker build -t andremariadevops/api-rocket:v6 .  

```

```bash

docker push andremariadevops/api-rocket:v6  

```

## Startup

Vamos configurar o startup probe . Para isso vamos alterar o arquivo `deployment.yaml`.

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: api-rocket

spec:
  replicas: 6
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
        image: andremariadevops/api-rocket:v6
        imagePullPolicy: IfNotPresent
        envFrom:
          - configMapRef:
              name: api-rocket
          - secretRef:
              name: api-rocket-secrets
        startupProbe:
          httpGet:
            path: /healthz
            port: 3000
          failureThreshold: 3
          successThreshold: 1
          timeoutSeconds: 1
          periodSeconds: 10
        resources:
          requests:
            cpu: 400m
            memory: 64Mi
          limits:
            cpu: "700m"
            memory: "128Mi"
        ports:
        - containerPort: 3000
```


#### Explicação do trecho `startupProbe` no Kubernetes

O trecho abaixo configura uma **`startupProbe`** em um arquivo de manifesto Kubernetes. Essa sonda é usada para verificar quando uma aplicação (ou contêiner) está **pronta durante a inicialização**, o que é útil para aplicativos que podem demorar a iniciar.

#### startupProbe
- **`startupProbe`** verifica a saúde do contêiner **durante o processo de inicialização**.
- Enquanto a sonda não for bem-sucedida, o Kubernetes considera que o contêiner **ainda não está pronto** para aceitar tráfego ou executar outras operações.

#### Componentes detalhados:

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

---

#### Funcionamento Geral
1. O Kubernetes verifica periodicamente o endpoint HTTP `/healthz` na porta 3000 assim que o contêiner começa a iniciar.
2. Caso o endpoint responda com sucesso (ex.: HTTP 200), o contêiner será considerado como inicializado.
3. Se o contêiner falhar 3 vezes consecutivas (por timeout ou erro no endpoint), ele será reiniciado.
4. A sonda é executada a cada 10 segundos, e o Kubernetes espera no máximo 1 segundo pela resposta em cada verificação.

Execute o comando :
```bash
kubectl apply -f k8s/deployment.yaml -n ns-rocket
```

vamos incluir um console.log na aplicação e vamos alterar o número da versão .

```typescript

import { Injectable } from '@nestjs/common';

@Injectable()
export class HealthService {

  checkHealth(): string {
    console.log("Checked app Healt");
    return 'Check Health Rocketset Api OK!';
  }

  checkReady(): string {
    console.log("Checked app Ready");
    return 'Check Ready Rocketset Api OK!';
  }
}


```

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: api-rocket

spec:
  replicas: 6
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
        image: andremariadevops/api-rocket:v7
        imagePullPolicy: IfNotPresent
        envFrom:
          - configMapRef:
              name: api-rocket
          - secretRef:
              name: api-rocket-secrets
        startupProbe:
          httpGet:
            path: /healthz
            port: 3000
          failureThreshold: 3
          successThreshold: 1
          timeoutSeconds: 1
          periodSeconds: 10
        resources:
          requests:
            cpu: 400m
            memory: 64Mi
          limits:
            cpu: "700m"
            memory: "128Mi"
        ports:
        - containerPort: 3000
```



Agora vamos excutar os seguintes comandos:

```bash

docker build -t andremariadevops/api-rocket:v7 .  

```

```bash

docker push andremariadevops/api-rocket:v7  

```

Execute o comando :
```bash
kubectl apply -f k8s/deployment.yaml -n ns-rocket
```

## Readiness

Vamos alterar o arquivo `deployment.yaml` adicionando as configurações do readiness.

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: api-rocket

spec:
  replicas: 6
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
        image: andremariadevops/api-rocket:v7
        imagePullPolicy: IfNotPresent
        envFrom:
          - configMapRef:
              name: api-rocket
          - secretRef:
              name: api-rocket-secrets
        startupProbe:
          httpGet:
            path: /healthz
            port: 3000
          failureThreshold: 3
          successThreshold: 1
          timeoutSeconds: 1
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /readyz
            port: 3000
          failureThreshold: 3
          successThreshold: 1
          timeoutSeconds: 1
          periodSeconds: 15
        resources:
          requests:
            cpu: 400m
            memory: 64Mi
          limits:
            cpu: "700m"
            memory: "128Mi"
        ports:
        - containerPort: 3000
```

Execute o comando :
```bash
kubectl apply -f k8s/deployment.yaml -n ns-rocket
```

![](image/Kubernetes/log-readiness.png)

Pensando em um cenário onde uma aplicação pode ter um alto tempo de **`"bootstrap"`**, as configurações atuais no nosso arquivo `deployment.yaml` não atende essa questão.

Logo vamos alterar ulguns pontos da nossa aplicação para termos esse cenário.

Em nossa aplicação vamos trabalhar no arquivo `main.ts`.

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

#### 1. Importação de módulos essenciais

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

```

#### NestFactory
- É uma classe fornecida pelo NestJS que facilita a criação de instâncias de aplicativos.
- `NestFactory.create()` é o método usado para criar a aplicação principal.

#### AppModule
- É o módulo raiz da aplicação. Ele geralmente serve como ponto de entrada, onde outros módulos, controladores e provedores são importados e configurados.
- Esse arquivo `app.module.ts` normalmente define os componentes principais que sua aplicação usará.

#### 2. Função bootstrap

```typescript
async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(process.env.PORT ?? 3000);
}
bootstrap();
```

#### Definição da função bootstrap
- A função bootstrap é uma função assíncrona que serve como o ponto de entrada para a aplicação.
- O NestJS é baseado em programação assíncrona devido à natureza do Node.js e seu loop de eventos.

#### Etapas detalhadas

#### Criar a aplicação:

```typescript
const app = await NestFactory.create(AppModule);
```

- Cria a aplicação principal baseada no módulo raiz (`AppModule`).
- Configura o pipeline de execução, middlewares, injeção de dependência e outras funcionalidades do NestJS.

#### 2. Iniciar o servidor HTTP:

```typescript
await app.listen(process.env.PORT ?? 3000);
```

- Aqui a aplicação é configurada para ouvir requisições HTTP.
- `process.env.PORT`:
  - Tenta usar a variável de ambiente `PORT` (caso ela esteja definida).
  - É útil para implantações onde o provedor de hospedagem (como Heroku ou AWS) define automaticamente a porta.
- `3000`:
  - Caso a variável `PORT` não esteja definida, o servidor usará a porta padrão 3000.

#### 3. Chamada da função

```typescript
bootstrap();
```

- Invoca a função bootstrap para iniciar a aplicação.

#### Fluxo geral do script

- O `NestFactory` cria a instância principal da aplicação utilizando o módulo raiz (`AppModule`).
- O servidor é configurado para escutar na porta definida (seja pela variável de ambiente ou pela porta padrão 3000).
- A aplicação é inicializada e começa a escutar requisições HTTP.

#### Contexto do NestJS

#### AppModule
- O NestJS usa o conceito de módulos, e o `AppModule` é o módulo inicial, onde você configura rotas, controladores, serviços, middlewares, etc.

#### Arquitetura modular
- O código segue o padrão arquitetural orientado a módulos, permitindo uma organização mais limpa e escalável.


Agora que entendemos como funciona, vamos alterar o script da seguinte maneira:

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(process.env.PORT ?? 3000);
}

setTimeout(() => {
  bootstrap();
}, 60000);

```
O código que você adicionou usa o setTimeout, que é uma função do JavaScript para executar um trecho de código após um determinado intervalo de tempo. Vamos ver o que ele faz no seu contexto:
Código completo:

```typescript
setTimeout(() => {
  bootstrap();
}, 60000);
```

**`"Explicação detalhada"`**:

- `"setTimeout(callback, delay)"`:
  setTimeout é uma função global do JavaScript que executa uma função (o callback) após um certo tempo (o delay em milissegundos).
  O primeiro parâmetro é uma função (neste caso, uma função de seta) que será executada quando o tempo de espera passar.
  O segundo parâmetro é o tempo de atraso em milissegundos (60000 milissegundos no seu caso, que corresponde a 60 segundos ou 1 minuto).

- `"bootstrap()"`:
  Dentro da função de callback, você está chamando a função `bootstrap()`, que, conforme vimos antes, é responsável por iniciar a aplicação NestJS.

- `"60000"`:
  Esse valor representa o tempo de atraso antes de a função `bootstrap()` ser chamada, e é dado em milissegundos.
  60000 milissegundos = 60 segundos = 1 minuto.

**`"O que acontece?"`**

    * Após 1 minuto de a aplicação ser carregada, a função `bootstrap()` será chamada novamente.
    
  * Isso significa que, após 60 segundos, a aplicação NestJS será reiniciada ou relançada, dependendo de como o seu código está configurado.

**`"Cenários comuns para usar esse padrão"`**:

    * Reinício ou retry: Esse padrão pode ser útil para aplicações que precisam tentar reiniciar em caso de falha ou para um "retry" após um tempo de espera.
    
  * Delay em inicialização: Você pode querer que a aplicação inicie somente após algum tempo de espera, talvez para aguardar a inicialização de serviços externos, como bancos de dados ou servidores.

**`"Possíveis implicações"`**:

    * Se a função `bootstrap()` inicializa um servidor `HTTP`, ao ser chamada novamente, o servidor será reiniciado. Isso pode ser problemático, pois um servidor `HTTP` não pode ser "reiniciado" enquanto ainda está rodando na mesma porta, a menos que você manipule explicitamente o processo (como matar o servidor antigo antes de criar um novo).
    
  * Cuidado com o uso excessivo de setTimeout em funções como essas, pois pode causar problemas de performance ou conflitos em servidores de produção, dependendo do contexto.

Se o seu objetivo for ter um comportamento específico (como tentar reiniciar após falha), seria interessante considerar outras abordagens mais controladas, como a utilização de mecanismos de "health check" ou "retry" mais robustos.

Agora vamos alterar o arquivo `health.services.ts`.

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class HealthService {

  checkHealth(): boolean {
    console.log("Checked app Healt");
    return (new Date().getMilliseconds() % 2 === 0);
  }

  checkReady(): string {
    console.log("Checked app Ready");
    return 'Check Ready Rocketset Api OK!';
  }
}

```

Agora vamos alterar o arquivo `health.controller.ts`.

```typescript
import { Controller, Get } from '@nestjs/common';
import { HealthService } from './health.service';

@Controller()
export class HealthController {
  constructor(private readonly healthService: HealthService) { }

  @Get('/healthz')
  healthz(): string {
    if (this.healthService.checkHealth()) {
      return 'Check Health Rocketset Api OK!';
    }
    throw new Error();
  }

  @Get('/readyz')
  readyz(): string {
    return this.healthService.checkReady();
  }
}

```

Essas alterações vão impactar o item[`startupProbe`]


Agora vamos excutar os seguintes comandos:

```bash

docker build -t andremariadevops/api-rocket:v8 .  

```

```bash

docker push andremariadevops/api-rocket:v8 

```

## Liveness

Vamos alterar o arquivo `deployment.yaml` adicionando as configurações do liveness.

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: api-rocket

spec:
  replicas: 6
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
        image: andremariadevops/api-rocket:v7
        imagePullPolicy: IfNotPresent
        envFrom:
          - configMapRef:
              name: api-rocket
          - secretRef:
              name: api-rocket-secrets
        startupProbe:
          httpGet:
            path: /healthz
            port: 3000
          failureThreshold: 3
          successThreshold: 1
          timeoutSeconds: 1
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /readyz
            port: 3000
          failureThreshold: 3
          successThreshold: 1
          timeoutSeconds: 1
          periodSeconds: 15
        livenessProbe:
          httpGet:
            path: /healthz
            port: 3000
          failureThreshold: 5
          successThreshold: 1
          periodSeconds: 10 
        resources:
          requests:
            cpu: 400m
            memory: 64Mi
          limits:
            cpu: "700m"
            memory: "128Mi"
        ports:
        - containerPort: 3000
```

Vamos subir a nossa alteração mas ainda não vamos usar a versão v8. Continuaremos com a versão v7.

Execute o comando :
```bash
kubectl apply -f k8s/deployment.yaml -n ns-rocket
```

![](image/Kubernetes/log-version-v7.png)

Agora vamos mudar para versão v8 ... `image: andremariadevops/api-rocket:v8`

![](image/Kubernetes/log-version-v8.png)

```bash
Startup probe failed: Get "http://10.244.1.43:3000/healthz": dial tcp 10.244.1.43:3000: connect: connection refused
```

Temos aqui um problema.

O startupProbe não está controlando o tempo, uma vez que está running o processo tenta fazer o teste.

A aplicação não está pronta para responder no momento da probe: O seu startupProbe tenta verificar se a aplicação está funcionando ao acessar /healthz na porta 3000. Porém, a aplicação pode não estar totalmente pronta para responder a esse pedido após o início.

Estamos usando o setTimeout de 60 segundos no código, para atrazar o início da aplicação, fazendo com que o Kubernetes não consiga se conectar a tempo.

Para solucionar vamos fazer um ajuste o startupProbe para ter um initialDelaySeconds maior para aguardar mais tempo antes da primeira tentativa de verificação.


```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: api-rocket

spec:
  replicas: 6
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
        image: andremariadevops/api-rocket:v8
        imagePullPolicy: IfNotPresent
        envFrom:
          - configMapRef:
              name: api-rocket
          - secretRef:
              name: api-rocket-secrets
        startupProbe:
          httpGet:
            path: /healthz
            port: 3000
          failureThreshold: 3
          successThreshold: 1
          timeoutSeconds: 1
          periodSeconds: 10
          initialDelaySeconds: 60
        readinessProbe:
          httpGet:
            path: /readyz
            port: 3000
          failureThreshold: 3
          successThreshold: 1
          timeoutSeconds: 1
          periodSeconds: 15
        livenessProbe:
          httpGet:
            path: /healthz
            port: 3000
          failureThreshold: 5
          successThreshold: 1
          timeoutSeconds: 1
          periodSeconds: 10 
        resources:
          requests:
            cpu: 400m
            memory: 64Mi
          limits:
            cpu: "700m"
            memory: "128Mi"
        ports:
        - containerPort: 3000
```

Execute o comando :
```bash
kubectl apply -f k8s/deployment.yaml -n ns-rocket
```

![](image/Kubernetes/log-error-pods.png)


Como podemos ver a aplicação está com muitos problemas .

## Refatorando a Aplicação e Entendendo Mais Sobre o Command

Vamos corrigir os problemas "desfazer as alterações relacionadas ao cenário de erro".

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class HealthService {

  checkHealth(): string {
    console.log("Checked app Healt");
    return 'Check Healt Rocketset Api OK!';
  }

  checkReady(): string {
    console.log("Checked app Ready");
    return 'Check Ready Rocketset Api OK!';
  }
}


import { Controller, Get } from '@nestjs/common';
import { HealthService } from './health.service';

@Controller()
export class HealthController {
  constructor(private readonly healthService: HealthService) { }

  @Get('/healthz')
  healthz(): string {
    return this.healthService.checkHealth();
  }

  @Get('/readyz')
  readyz(): string {
    return this.healthService.checkReady();
  }
}

```

Agora vamos excutar os seguintes comandos:

```bash

docker build -t andremariadevops/api-rocket:v9 .  

```

```bash

docker push andremariadevops/api-rocket:v9 

```

Agora vmos atualizar a versão do deployment

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: api-rocket

spec:
  replicas: 6
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
        image: andremariadevops/api-rocket:v9
        imagePullPolicy: IfNotPresent
        envFrom:
          - configMapRef:
              name: api-rocket
          - secretRef:
              name: api-rocket-secrets
        startupProbe:
          httpGet:
            path: /healthz
            port: 3000
          failureThreshold: 3
          successThreshold: 1
          timeoutSeconds: 1
          periodSeconds: 10
          initialDelaySeconds: 10
        readinessProbe:
          httpGet:
            path: /readyz
            port: 3000
          failureThreshold: 3
          successThreshold: 1
          timeoutSeconds: 1
          periodSeconds: 15
          initialDelaySeconds: 5
        livenessProbe:
          httpGet:
            path: /healthz
            port: 3000
          failureThreshold: 2
          successThreshold: 1
          timeoutSeconds: 1
          periodSeconds: 10 
          initialDelaySeconds: 5
        resources:
          requests:
            cpu: 400m
            memory: 64Mi
          limits:
            cpu: "700m"
            memory: "128Mi"
        ports:
        - containerPort: 3000
```

Execute o comando :
```bash
kubectl apply -f k8s/deployment.yaml -n ns-rocket
```

![](image/Kubernetes/log-ok-pods.png)

## Garantindo prontidão da aplicação


```bash
kubectl get svc -n ns-rocket
```

```bash
PS C:\Repo\rocketseat.ci.api> kubectl get svc -n ns-rocket
NAME             TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
api-rocket-svc   ClusterIP   10.96.107.209   <none>        80/TCP    14d
PS C:\Repo\rocketseat.ci.api> 
```

```bash
kubectl describe svc api-rocket-svc -n ns-rocket
```

```bash
PS C:\Repo\rocketseat.ci.api> kubectl describe svc api-rocket-svc -n ns-rocket
Name:              api-rocket-svc
Namespace:         ns-rocket
Labels:            <none>
Annotations:       <none>
Selector:          api=api-rocket
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.96.107.209
IPs:               10.96.107.209
Port:              <unset>  80/TCP
TargetPort:        3000/TCP
Endpoints:         10.244.1.45:3000,10.244.1.46:3000,10.244.1.47:3000 + 3 more...
Session Affinity:  None
Events:            <none>
```

![](image/Kubernetes/network-service-endpoint.png)

![](image/Kubernetes/network-service-endpoint-pods.png)


Nossa aplicação só recebe tráfico se passar pelas etapas **`"startupProbe"`** e **`"readinessProbe"`** 


## Alternativas na camada da aplicação


