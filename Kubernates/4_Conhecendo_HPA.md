# Conhecendo o HPA

## O que é Escala

## Conhecendo a escala vertical

**`"Escala Vertical"`** 
- Aumenta o tamanho da máquina - em termos de cpu, memória, armazenamento.
- Necessário em casos de grande escala com apenas uma máquina.
- Menor redundância e maior risco de indisponibilidade.
- Limite do próprio hardware
- Promove down time dentro de um determinado período
- VPA dentro do k8s

## Explorando a escala horizontal

**`"Escala horizontal"`** 
- Replicação da infraestrutura.
- Visa Distribuir igualmente as cargas de trabalho.
- Aumento a tolerância a falhas.
- Por ser horizontal, temos uma maior redundância.
- A escala não está restrita aos limites do hardware.
- Condicionada aos recursos e/ou eventos - KEDA.
- ! Possíveis problemas na consistência de dados!

## Como o Metrics-Server funciona

O **`"Metrics-Server"`**  é um componente essencial no ecossistema do Kubernetes para coletar e expor métricas de uso de recursos `(como CPU e memória)` em tempo real dos nós e dos pods em um cluster. 

Ele é um substituto moderno para o antigo heapster e faz parte das APIs de métricas do Kubernetes.

---

#### **Como funciona o Metrics Server**

1. **Coleta de Métricas:**
   - O Metrics Server coleta métricas de uso de recursos diretamente dos agentes do Kubernetes, chamados de **kubelet**, que estão presentes em cada nó do cluster.
   - O kubelet usa o **cAdvisor** (integrado) para monitorar o uso de recursos no nível do nó e dos contêineres.

2. **Exposição de Métricas:**
   - As métricas coletadas pelo Metrics Server são expostas por meio da **API de métricas do Kubernetes** (`metrics.k8s.io`).
   - Essas métricas são disponibilizadas para componentes internos, como o **Horizontal Pod Autoscaler (HPA)**, e também podem ser acessadas por comandos como `kubectl top`.

3. **Eficiência e Escalabilidade:**
   - Ele é projetado para ser leve e eficiente, armazenando apenas métricas em tempo real (não mantém histórico).
   - É ideal para aplicações onde o foco está em ações imediatas baseadas em métricas, como escalonamento horizontal.

---

#### **Para que serve o Metrics Server**

1. **Escalonamento Horizontal Automático (HPA):**
   - Permite que o Kubernetes ajuste automaticamente o número de réplicas de um deployment ou replica set com base em métricas de uso de CPU e memória.

2. **Monitoramento em Tempo Real:**
   - Fornece informações sobre o consumo atual de recursos dos pods e nós do cluster.

## Adicionando o Metrics-Server no nosso Cluster"

Vamos acessar ao site :

https://github.com/kubernetes-sigs/metrics-server

Vamos rodar o seguinte comando :

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

![](image/Kubernetes/metrics-server-command.png)

Vamos ver se funciona.

```bash
kubectl get po -n kube-system
```

![](image/Kubernetes/command-kubctl-get-po-metrics-server.png)

Como podemos ver na imagem o pod `metrics-server-54bf7cdd6-z7bvj` não esta em execução.

Para analizarmos melhor o problema vamos rodar o seguinte comando:

```bash
kubectl logs metrics-server-54bf7cdd6-z7bvj -n kube-system
```

```bash
E1203 22:07:33.612329       1 scraper.go:149] "Failed to scrape node" err="Get \"https://172.23.0.2:10250/metrics/resource\": tls: failed to verify certificate: x509: cannot validate certificate for 172.23.0.2 because it doesn't contain any IP SANs" node="secund-cluster-rocketseat-worker"
E1203 2
```

Como estamos em um ambiente local não é possível validar o certificado.

Para resolver esse problema vamos ao site https://github.com/kubernetes-sigs/metrics-server 

![](image/Kubernetes/solve-problema-certification.png)

- primeiro é excluir o metrics-server

```bash
kubectl delete -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

- Vamos acessar a pasta k8s e vamos rodar o seguinte comando:

```bash
wget https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

ou 

```bash
Invoke-WebRequest -Uri https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml -OutFile components.yaml

```

Aqui fizemos o download para a pasta para internalizar o arquivo para que possamos configura-lo conforme a necessidade.

![](image/Kubernetes/new-file.yaml)

Agora vamos editar o conteudo do arquivo.

- renomear o arquivo para `metrics-server.yaml`
- vamos adicionar `--kubelet-insecure-tls` no trecho `- args` do trecho `Deployment` do arquivo.

![](image/Kubernetes/add-kubelet-insecure-tls.png)

- vamos rodar o camando

```bash
kubectl apply -f metrics-server.yaml
```

```bash
kubectl get po -n kube-system
```

![](image/Kubernetes/log-ok-metrics-server.png)

agora podemos ver que as metricas nos pods relacionados a nossa api possuem valores:

![](image/Kubernetes/pods-metricris-log.png)

- Consultando logs pleo comando:

```bash
kubectl top po -n ns-rocket
```

![](image/Kubernetes/logs-top-pods.png)

## Entendendo os Principais Triggers

## Explorando a V1 do HPA

Vamos criar uma novo arquivo `hpa.yaml`

```yaml
apiVersion: autoscaling/v1
kind: HorizontalPodAutoscaler

metadata:
  name: api-rocket-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-rocket
  minReplicas: 3
  maxReplicas: 8
  targetCPUUtilizationPercentage: 75

```

![](image/Kubernetes/new-config-file-hpa.png)


```bash
kubectl apply -f k8s/hpa.yaml -n ns-rocket
```

```bash
kubectl get hpa -n ns-rocket
```

![](image/Kubernetes/overview-hpa-start.png)

Aqui estamos usando a v1 para o hpa. Vamos fazer o upgrade para V2. Mas antes vamos excluir o hpa do cluster.

```bash
kubectl delete -f k8s/hpa.yaml -n ns-rocket
```
Vamos renomear o arquivo para `hpa-v1.yaml`

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler

metadata:
  name: api-rocket-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-rocket
  minReplicas: 3
  maxReplicas: 8
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 75
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```


#### Explicação do Script Horizontal Pod Autoscaler (HPA)

Este script define um **HorizontalPodAutoscaler (HPA)** no Kubernetes, que ajusta automaticamente o número de réplicas de pods em um deployment com base no uso de recursos (CPU e memória). Vamos analisar o script em detalhes:

---

#### **Cabeçalho**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
```

**`apiVersion: autoscaling/v2`**: Especifica a versão da API do HPA. A versão v2 suporta métricas avançadas e configuráveis, como diferentes tipos de recursos e thresholds personalizados.

**`kind: HorizontalPodAutoscaler`**: Declara que o objeto a ser criado é um HPA.

#### **Metadados**

```yaml
metadata:
  name: api-rocket-hpa
```
**`name: api-rocket-hpa`**: Define o nome do recurso HPA no cluster. Esse nome será usado para identificá-lo.

#### Especificações

```yaml
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-rocket
```

#### 1. Referência ao Deployment

**`scaleTargetRef`**: Especifica o recurso que será escalado automaticamente.
- **`apiVersion: apps/v1`**: A API para gerenciar Deployment no Kubernetes.
- **`kind: Deployment`**: Indica que o HPA se aplica a um Deployment.
- **`name: api-rocket`**: O nome do Deployment que será escalado.

#### 2. Configuração de Réplicas

```yaml
  minReplicas: 3
  maxReplicas: 8
```

**`minReplicas: 3`**: O número mínimo de réplicas garantidas para o deployment. Nunca haverá menos de 3 réplicas rodando, mesmo com baixo uso de recursos.

**`maxReplicas: 8`**: O número máximo de réplicas permitido. Se a carga ultrapassar este limite, o Kubernetes não criará mais réplicas.

#### 3. Métricas de Escalonamento

```yaml
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 75
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```


#### Métrica 1: CPU

**`type: Resource`**: Indica que o escalonamento será baseado no uso de recursos do pod.

**`name: cpu`**: Especifica que a métrica monitorada será o uso de CPU.

**`target`**:

- **`type: Utilization`**: Define que a métrica será avaliada como porcentagem de uso em relação ao limite configurado no pod.

- **`averageUtilization`**: 75: Se o uso médio de CPU ultrapassar 75% em todas as réplicas, o HPA criará mais réplicas (respeitando os limites definidos).

#### Métrica 2: Memória

**`name: memory`**: Especifica que a métrica monitorada será o uso de memória.

**`averageUtilization`**: 80: Se o uso médio de memória ultrapassar 80%, o HPA criará mais réplicas para lidar com a carga.

#### Como Funciona na Prática

Monitoramento Contínuo: O HPA monitora o uso de CPU e memória em todas as réplicas do deployment api-rocket.

Ajuste Automático:
- Se o uso médio de CPU for maior que 75% ou o uso médio de memória for maior que 80%, o HPA escala o número de réplicas até um máximo de 8.

- Se o uso médio estiver abaixo dos thresholds e houver mais de 3 réplicas, o HPA reduz o número de réplicas, respeitando o mínimo de 3.

Eficiência de Recursos: Esse comportamento garante que o cluster mantenha um desempenho estável e evite sobrecargas ou subutilização.

#### Resumo

Este script define uma política de escalonamento automático para o deployment api-rocket, ajustando dinamicamente o número de réplicas entre 3 e 8, com base no uso de CPU (75%) e memória (80%). 

Ele é útil para garantir alta disponibilidade e eficiência, especialmente em cargas de trabalho variáveis.



```bash
kubectl apply -f k8s/hpa.yaml -n ns-rocket
```

```bash
kubectl get hpa -n ns-rocket
```

```bash
PS C:\Repo\rocketseat.ci.api> kubectl get hpa -n ns-rocket
NAME             REFERENCE               TARGETS                        MINPODS   MAXPODS   REPLICAS   AGE
api-rocket-hpa   Deployment/api-rocket   cpu: 0%/75%, memory: 79%/80%   3         8         3          36s
PS C:\Repo\rocketseat.ci.api> 

```

## Criando o HPA Utilizando a V2

Vamos criar uma novo arquivo `hpa.yaml`

```yaml

apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler

metadata:
  name: api-rocket-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-rocket
  minReplicas: 3
  maxReplicas: 8
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 75
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  

```

```bash
kubectl apply -f k8s/hpa-v2.yaml -n ns-rocket
```

```bash
kubectl get hpa -n ns-rocket
```

## Estressando a Nossa Aplicação


Vamos testar a ideia de `HorizontalPodAutoscaler` estressando a aplicação.

Para isso vamos acessar o site : https://github.com/fortio/fortio

Aqui temos a possibilidade de realizar testes de carga.

```bash
kubectl run -it fortio -n ns-rocket --rm --image=fortio/fortio -- load -qps 6000 -t 120s -c 50 "http://api-rocket-svc/example-k8s"
```
#### Descrição do Comando

Este comando executa um **teste de carga** em um serviço web hospedado no Kubernetes usando a ferramenta **Fortio**. Abaixo está o detalhamento de cada parte do comando.

---

#### Partes do Comando

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

#### Resumo do que o comando faz

1. Cria um pod temporário no namespace **`ns-rocket`** utilizando a imagem do Fortio.
2. Executa um teste de carga contra o endpoint **`http://api-rocket-svc/example-k8s`**.
3. Envia **6.000 requisições por segundo** durante **120 segundos**, utilizando **50 conexões simultâneas**.
4. Após o término do teste, o pod é automaticamente removido.

---

#### Resultados Esperados

Durante o teste, o Fortio coleta as seguintes métricas:
- **Latência das requisições** (ex.: p95, p99).
- **Taxa de sucesso** das requisições.
- **Erros de rede** ou falhas no serviço.


```bash

Connection time histogram (s) : count 50 avg 0.00077979046 +/- 0.002684 min 4.5061e-05 max 0.015008246 sum 0.038989523
# range, mid point, percentile, count
>= 4.5061e-05 <= 0.001 , 0.000522531 , 94.00, 47
> 0.004 <= 0.005 , 0.0045 , 96.00, 1
> 0.011 <= 0.012 , 0.0115 , 98.00, 1
> 0.014 <= 0.0150082 , 0.0145041 , 100.00, 1
# target 50% 0.00054329
# target 75% 0.000802784
# target 90% 0.000958481
# target 99% 0.0145041
# target 99.9% 0.0149578
Sockets used: 50 (for perfect keepalive, would be 50)
Uniform: false, Jitter: false, Catchup allowed: true
IP addresses distribution:
10.96.107.209:80: 50
Code 200 : 165015 (100.0 %)
Response Header Sizes : count 165015 avg 228 +/- 0 min 228 max 228 sum 37623420
Response Body/Total Sizes : count 165015 avg 298 +/- 0 min 298 max 298 sum 49174470
All done 165015 calls (plus 50 warmup) 36.354 ms avg, 1374.1 qps
Session ended, resume using 'kubectl attach fortio -c fortio -i -t' command when the pod is running
pod "fortio" deleted

```

## Explorando mais cenários de estresse

Vamos alterar o arquivo `app.service.ts` da aplicação :

```typescript
import { Injectable } from '@nestjs/common';
import  { createWriteStream } from 'fs'
@Injectable()
export class AppService {
  getHello(): string {
    console.log(`secret: ${ process.env.API_KEY }`)
    return 'Rocketset Api!';
  }

  getExample(): string {
    const file = createWriteStream('rocketseat.txt')
    for (let index = 0; index <= 10000; index++) {
      file.write('Estou escrevendo em um arquivo\n');
    }
    file.end();
    return `Estou rodando no K8s! ${ Date.UTC }: ${process.env.APP}`;
  }
}
```

```bash
docker build -t andremariadevops/api-rocket:v5 .    
```

```bash

docker push andremariadevops/api-rocket:v5  

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
        image: andremariadevops/api-rocket:v5
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

```bash
 kubectl apply -f k8s/deployment.yaml -n ns-rocket 
```

Agora vamos iniciar o nosso teste de estesse

```bash
kubectl run -it fortio -n ns-rocket --rm --image=fortio/fortio -- load -qps 6000 -t 120s -c 50 "http://api-rocket-svc/example-k8s"
```

```bash
Connection time histogram (s) : count 252 avg 0.00070557625 +/- 0.003099 min 4.8129e-05 max 0.03114827 sum 0.177805214
# range, mid point, percentile, count
>= 4.8129e-05 <= 0.001 , 0.000524065 , 95.63, 241
> 0.001 <= 0.002 , 0.0015 , 96.83, 3
> 0.002 <= 0.003 , 0.0025 , 97.62, 2
> 0.003 <= 0.004 , 0.0035 , 98.02, 1
> 0.004 <= 0.005 , 0.0045 , 98.41, 1
> 0.006 <= 0.007 , 0.0065 , 98.81, 1
> 0.025 <= 0.03 , 0.0275 , 99.60, 2
> 0.03 <= 0.0311483 , 0.0305741 , 100.00, 1
# target 50% 0.000543895
# target 75% 0.000793761
# target 90% 0.000943681
# target 99% 0.0262
# target 99.9% 0.0308589
Sockets used: 252 (for perfect keepalive, would be 50)
Uniform: false, Jitter: false, Catchup allowed: true
IP addresses distribution:
10.96.107.209:80: 252
Code  -1 : 120 (2.8 %)
Code 200 : 4159 (97.2 %)
Response Header Sizes : count 4279 avg 221.60598 +/- 37.64 min 0 max 228 sum 948252
Response Body/Total Sizes : count 4279 avg 289.64291 +/- 49.2 min 0 max 298 sum 1239382
All done 4279 calls (plus 50 warmup) 1413.665 ms avg, 34.9 qps
Session ended, resume using 'kubectl attach fortio -c fortio -i -t' command when the pod is running
pod "fortio" deleted
```

## Alterando Recursos e Réplicas da Aplicação

NO ultimo test temos como resultado ***`"All done 4279 calls (plus 50 warmup) 1413.665 ms avg, 34.9 qps"`***

vamos altera o arquivo `hpa-v2.yaml` 

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler

metadata:
  name: api-rocket-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-rocket
  minReplicas: 6
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 75
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  
```

vamos altera o arquivo `deployment.yaml` 

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
        image: andremariadevops/api-rocket:v5
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
            cpu: 400m
            memory: 64Mi
          limits:
            cpu: "700m"
            memory: "128Mi"
        ports:
        - containerPort: 3000
```

```bash
kubectl run -it fortio -n ns-rocket --rm --image=fortio/fortio -- load -qps 6000 -t 120s -c 50 "http://api-rocket-svc/example-k8s"
```

```bash

Connection time histogram (s) : count 51 avg 0.024701479 +/- 0.03977 min 8.0373e-05 max 0.100979221 sum 1.25977541
# range, mid point, percentile, count
>= 8.0373e-05 <= 0.001 , 0.000540186 , 52.94, 27
> 0.001 <= 0.002 , 0.0015 , 56.86, 2
> 0.004 <= 0.005 , 0.0045 , 58.82, 1
> 0.005 <= 0.006 , 0.0055 , 66.67, 4
> 0.006 <= 0.007 , 0.0065 , 70.59, 2
> 0.016 <= 0.018 , 0.017 , 72.55, 1
> 0.03 <= 0.035 , 0.0325 , 74.51, 1
> 0.035 <= 0.04 , 0.0375 , 76.47, 1
> 0.04 <= 0.045 , 0.0425 , 78.43, 1
> 0.08 <= 0.09 , 0.085 , 80.39, 1
> 0.09 <= 0.1 , 0.095 , 84.31, 2
> 0.1 <= 0.100979 , 0.10049 , 100.00, 8
# target 50% 0.000946945
# target 75% 0.03625
# target 90% 0.100355
# target 99% 0.100917
# target 99.9% 0.100973
Sockets used: 51 (for perfect keepalive, would be 50)
Uniform: false, Jitter: false, Catchup allowed: true
IP addresses distribution:
10.96.107.209:80: 51
Code 200 : 10003 (100.0 %)
Response Header Sizes : count 10003 avg 228 +/- 0 min 228 max 228 sum 2280684
Response Body/Total Sizes : count 10003 avg 298 +/- 0 min 298 max 298 sum 2980894
All done 10003 calls (plus 50 warmup) 600.651 ms avg, 82.9 qps
Session ended, resume using 'kubectl attach fortio -c fortio -i -t' command when the pod is running
pod "fortio" deleted
```

***`"All done 4279 calls (plus 50 warmup) 1413.665 ms avg, 34.9 qps"`***
***`"All done 10003 calls (plus 50 warmup) 600.651 ms avg, 82.9 qps"`***

Como podemos ver, o teste foi executado em um desempenho superior.

## Definindo tempo de reação para escalonar a quantidade de réplicas

Vamos alterar o arquivo `hpa-v2.yaml`

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler

metadata:
  name: api-rocket-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-rocket
  minReplicas: 6
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 75
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
        stabilizationWindowSeconds: 30
        policies:
          - type: Pods
            value: 2
            periodSeconds: 15
        #   - type: Percent
        #     value: 20
        #     periodSeconds: 15
        # selectPolicy : Max
```


#### Comportamento do HorizontalPodAutoscaler (HPA)

O trecho `behavior` define o comportamento personalizado de escalonamento do **HorizontalPodAutoscaler (HPA)** ao aumentar ou diminuir o número de réplicas dos pods em um cluster Kubernetes. Vamos analisar cada parte desse trecho:

---

#### Seção: behavior

Essa seção é opcional e permite configurar políticas específicas para **escalonamento para cima** (`scaleUp`) ou **escalonamento para baixo** (`scaleDown`), fornecendo maior controle sobre como e quando o HPA deve realizar essas ações.

---

#### scaleDown (Escalonamento para baixo)

Aqui, configura-se como o HPA reduzirá o número de réplicas dos pods.

#### stabilizationWindowSeconds: 30

- Especifica uma "janela de estabilização" de **30 segundos**.
- Isso significa que o HPA esperará **30 segundos** antes de decidir reduzir o número de réplicas após detectar que o uso de recursos está abaixo do alvo.
- Serve para evitar que o HPA oscile frequentemente entre diferentes números de réplicas em situações de carga variável (chamado de *flapping*).

---

#### policies

Define as políticas para limitar como o **escalonamento para baixo** pode ocorrer. Neste exemplo, há apenas uma política ativa:

- **type: Pods**
  - Essa política controla diretamente o número de réplicas que podem ser reduzidas em uma única ação.
  - **value: 2**: Especifica que, no máximo, o HPA pode reduzir **2 réplicas** de uma só vez.
  - **periodSeconds: 15**: Especifica que essa redução máxima (**2 réplicas**) pode ocorrer a cada **15 segundos**.

---

#### Comentário

As linhas comentadas indicam que o HPA poderia usar outra política (baseada em **porcentagem**), mas ela foi desativada. Se a política `Percent` fosse ativada:

- **type: Percent**: O HPA reduziria as réplicas com base em uma porcentagem do número total de pods em execução.
- **value: 20**: Reduziria no máximo **20%** dos pods em execução.

#### selectPolicy

Define qual política usar se houver várias aplicáveis. Por exemplo:

- **Max**: Escolhe a política que resulta na maior redução (mais agressiva).
- **Min**: Escolhe a política que resulta na menor redução (mais conservadora).

⚠️ Apenas uma política pode ser aplicada por vez, e o `selectPolicy` especifica qual deve ser priorizada se houver múltiplas políticas ativas.


#### Explicação do Trecho `behavior` para `scaleUp` no HorizontalPodAutoscaler (HPA)

O trecho `behavior` configura o comportamento do **HorizontalPodAutoscaler (HPA)** para escalonamento **para cima** (`scaleUp`), ou seja, aumentar o número de réplicas dos pods quando há aumento na carga. Abaixo está uma explicação detalhada:

#### Trecho `scaleUp`

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler

metadata:
  name: api-rocket-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: api-rocket
  minReplicas: 6
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 75
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
        stabilizationWindowSeconds: 30
        policies:
          - type: Pods
            value: 2
            periodSeconds: 15
        #   - type: Percent
        #     value: 20
        #     periodSeconds: 15
        # selectPolicy : Max
    scaleUp:
        stabilizationWindowSeconds: 5
        policies:
          - type: Pods
            value: 2
            periodSeconds: 5
```
#### Seção: `scaleUp`

Essa seção define como o HPA irá aumentar o número de réplicas do Deployment.

#### `stabilizationWindowSeconds: 5`
- Especifica uma janela de estabilização de **5 segundos**.
- Isso significa que o HPA esperará **5 segundos** antes de aumentar o número de réplicas, mesmo que as métricas indiquem a necessidade de escalonamento.
- Esse intervalo reduz oscilações rápidas no número de réplicas, garantindo que o aumento seja consistente.

#### `policies`
Define as políticas que limitam como o escalonamento para cima ocorrerá. Aqui, há apenas uma política ativa:

- **`type: Pods`**
  - Define que o aumento será baseado no número absoluto de réplicas.
  - **`value: 2`**: O HPA pode aumentar no máximo **2 réplicas** por vez.
  - **`periodSeconds: 5`**: Esse aumento de 2 réplicas pode ocorrer no máximo a cada **5 segundos**.

---

#### Como Funciona o Escalonamento para Cima

Quando o HPA detecta que a carga em um Deployment ultrapassou os limites definidos nas métricas (como uso de CPU ou memória), ele inicia o processo de escalonamento para cima. O comportamento é configurado da seguinte maneira:

#### Janela de Estabilização (`stabilizationWindowSeconds`)
- Serve para prevenir oscilações rápidas no número de réplicas causadas por variações momentâneas na carga.

#### Política Baseada em Réplicas Absolutas (`type: Pods`)
- Permite um aumento controlado no número de réplicas.
- Por exemplo, se há **4 réplicas** atualmente e a carga exige mais réplicas, o HPA só adicionará **até 2 réplicas** em intervalos de **5 segundos**, mesmo que mais réplicas sejam necessárias.

---

#### Vantagens do Trecho Configurado

#### **Controle Fino do Escalonamento**
- Evita um aumento excessivo e rápido no número de réplicas, o que pode levar ao uso ineficiente de recursos.
- Permite um ajuste gradual e controlado.

#### **Estabilidade**
- Reduz oscilações no número de réplicas, mesmo em cenários de carga variável.

#### **Respostas Rápidas**
- A janela de estabilização curta (**5 segundos**) permite respostas quase imediatas em situações de alta demanda.


Agora vamos testar a alteração rodando o comando 

```bash
kubectl apply -f k8s/hpa-v2.yaml -n ns-rocket
```


Agora podemos dora novamente o teste de estresse:

```bash
kubectl run -it fortio -n ns-rocket --rm --image=fortio/fortio -- load -qps 6000 -t 120s -c 50 "http://api-rocket-svc/example-k8s"
```

```bash
Connection time histogram (s) : count 57 avg 0.0013138895 +/- 0.00426 min 7.8184e-05 max 0.02913538 sum 0.074891704
# range, mid point, percentile, count
>= 7.8184e-05 <= 0.001 , 0.000539092 , 85.96, 49
> 0.001 <= 0.002 , 0.0015 , 92.98, 4
> 0.003 <= 0.004 , 0.0035 , 94.74, 1
> 0.004 <= 0.005 , 0.0045 , 96.49, 1
> 0.014 <= 0.016 , 0.015 , 98.25, 1
> 0.025 <= 0.0291354 , 0.0270677 , 100.00, 1
# target 50% 0.000606308
# target 75% 0.000879972
# target 90% 0.001575
# target 99% 0.0267782
# target 99.9% 0.0288997
Sockets used: 57 (for perfect keepalive, would be 50)
Uniform: false, Jitter: false, Catchup allowed: true
IP addresses distribution:
10.96.107.209:80: 57
Code 200 : 10411 (100.0 %)
Response Header Sizes : count 10411 avg 228 +/- 0 min 228 max 228 sum 2373708
Response Body/Total Sizes : count 10411 avg 298 +/- 0 min 298 max 298 sum 3102478
All done 10411 calls (plus 50 warmup) 579.522 ms avg, 85.7 qps
Session ended, resume using 'kubectl attach fortio -c fortio -i -t' command when the pod is running
pod "fortio" deleted
```

Comparando o resultado com as outras execuções de teste de estresse temos:

***`"All done 4279 calls (plus 50 warmup) 1413.665 ms avg, 34.9 qps"`***
***`"All done 10003 calls (plus 50 warmup) 600.651 ms avg, 82.9 qps"`***

***`"All done 10411 calls (plus 50 warmup) 579.522 ms avg, 85.7 qps"`***


