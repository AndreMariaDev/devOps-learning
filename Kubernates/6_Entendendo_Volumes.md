# Entendendo mais sobre volumes

## Volumes e StorageClass
No Kubernetes, volumes são recursos utilizados para armazenar dados persistentes que podem ser acessados por contêineres, 
mesmo que estes sejam reiniciados ou recriados. 

O ideal é que a aplicação seja efemera.

O `StorageClass` é um recurso que define como o armazenamento deve ser provisionado dinamicamente no cluster, 
especificando o tipo de armazenamento (como SSD ou HDD) e outros parâmetros específicos do provedor de armazenamento.

Vamos aprofundar sobre esse item :

```bash
kubectl get storageclass
```

```bash
PS C:\Repo\rocketseat.ci.api> kubectl get storageclass
NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   fa
```

#### Explicação: Storage Class no Kubernetes

Abaixo está a explicação detalhada dos campos exibidos na saída:

| Campo               | Valor                     | Descrição                                                                                                                                                              |
|---------------------|---------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **NAME**            | `standard (default)`      | Nome da *Storage Class*. O `(default)` indica que esta é a classe de armazenamento padrão do cluster. Se nenhum PVC especificar uma *Storage Class*, usará esta.      |
| **PROVISIONER**     | `rancher.io/local-path`   | Provisionador responsável por criar os volumes. No caso, o `rancher.io/local-path` utiliza o **Local Path Provisioner**, que armazena os volumes localmente nos nós. |
| **RECLAIMPOLICY**   | `Delete`                 | Define o que acontece com o volume após a exclusão do PVC:                                                                                                           |
|                     |                           | - **Delete**: O volume é automaticamente excluído junto com o PVC.                                                                                                   |
|                     |                           | - **Retain**: O volume é mantido, e o gerenciamento passa a ser manual.                                                                                             |
| **VOLUMEBINDINGMODE**| `WaitForFirstConsumer`   | Controla quando e onde o volume será provisionado:                                                                                                                   |
|                     |                           | - **WaitForFirstConsumer**: Provisiona o volume apenas quando um pod usando o PVC associado é agendado em um nó, garantindo o provisionamento correto.              |
|                     |                           | - **Immediate**: Provisiona o volume assim que o PVC é criado, sem esperar pelo agendamento do pod.                                                                 |
| **ALLOWVOLUMEEXPANSION** | `false`             | Indica se a expansão de volume é permitida:                                                                                                                          |
|                     |                           | - **true**: Permite aumentar o tamanho do volume após a criação.                                                                                                    |
|                     |                           | - **false**: Não permite alterações no tamanho do volume.                                                                                                           |                                                                                    |

#### Resumo

A *Storage Class* chamada `standard` possui as seguintes características:
- Usa o **Local Path Provisioner** (`rancher.io/local-path`).
- Configurada com a política de retenção **Delete**, excluindo os volumes automaticamente quando o PVC correspondente for excluído.
- O volume só é provisionado quando um pod que utiliza o PVC associado é agendado (**WaitForFirstConsumer**).
- **Não permite expansão** de volumes.


Essa configuração é comum em clusters Kubernetes simples, como aqueles criados com ferramentas como **Rancher** ou **K3s**, que utilizam armazenamento local para volumes persistentes.


## PersistentVolume (PV)
Um `PersistentVolume` é um recurso no Kubernetes que representa uma unidade de armazenamento configurada de forma independente 
do ciclo de vida dos pods. 

Ele pode ser provisionado de forma estática pelo administrador ou dinamicamente por meio de um `StorageClass`.
(O PV e que interage com o `StorageClass`)

Os PVs possuem especificações como capacidade, acesso e configurações do backend de armazenamento.
(Aloca um espaço de armazenamento)

## PersistentVolumeClaim (PVC)
Um `PersistentVolumeClaim` é uma solicitação feita por um pod para utilizar um `PersistentVolume`. 

Ele define as especificações de armazenamento necessárias, como tamanho e modos de acesso (leitura/escrita). 

O Kubernetes faz o mapeamento automático entre PVCs e PVs que atendem às necessidades especificadas.

Persistent Volume Claim “reinvindicar” (verbo to claim, em inglês)

Para utilizar um PVC em um pod ou em um deployment, ele deve ser montado como um volume. 

Exemplo:

StorageClass => Valume : Persistent - 10Gi => Clain - pvc - 1Gi

![](image/Kubernetes/diagrama_pv.png)


## Criando o StorageClass
O `StorageClass` é criado por meio de um manifesto YAML. 

Ele especifica o provedor de armazenamento e suas propriedades. 

```bash
PS C:\Repo\rocketseat.ci.api> kubectl get storageclass
NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  22d
PS C:\Repo\rocketseat.ci.api> 
```


Exemplo de criação:

Vamos criar um arquivo na pasta k8s `k8s/storageclass.yaml`

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: api-rocket-storage-class
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
```

```bash
kubectl apply -f k8s/storageclass.yaml
```

## Reservando o espaço com o PV
O `PersistentVolume` é criado manualmente ou automaticamente com base em um `StorageClass`. 

Exemplo de manifesto YAML para um PV:
Vamos criar um arquivo na pasta k8s `k8s/persistentvolume.yaml`


```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: api-rocket-pv
  labels:
    name: api-rocket-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Delete
  storageClassName: standard
  hostPath:
    path: "/mnt/data"
```

Este exemplo reserva 2 GiB de armazenamento em um caminho do sistema de arquivos do host.

```bash
kubectl apply -f k8s/persistentvolume.yaml
```

```bash
PS C:\Repo\rocketseat.ci.api> kubectl get pv
NAME            CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   VOLUMEATTRIBUTESCLASS   REASON   AGE
api-rocket-pv   2Gi        RWO            Delete           Available           standard       <unset>                          19s
PS C:\Repo\rocketseat.ci.api>

```

## Criando o PVC
O `PersistentVolumeClaim` é criado para solicitar um volume com base nas especificações desejadas. 

Exemplo de manifesto YAML para um PVC:

Vamos criar um arquivo na pasta k8s `k8s/persistentvolumeclaim.yaml`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: api-rocket-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 512Mi
  storageClassName: standard
  selector:
    matchLabels:
      name:  api-rocket-pv
```

```bash
kubectl apply -f k8s/persistentvolumeclaim.yaml -n ns-rocket
```

Este PVC solicita um volume com 10 GiB de armazenamento utilizando o `StorageClass` especificado.

```bash
PS C:\Repo\rocketseat.ci.api> kubectl describe pvc -n ns-rocket
Name:          api-rocket-pvc
Namespace:     ns-rocket
StorageClass:  standard
Status:        Pending
Volume:        
Labels:        <none>
Annotations:   <none>
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      
Access Modes:  
VolumeMode:    Filesystem
Used By:       <none>
Events:
  Type    Reason                Age               From                         Message
  ----    ------                ----              ----                         -------
  Normal  WaitForFirstConsumer  4s (x7 over 91s)  persistentvolume-controller  waiting for first consumer to be created before binding
PS C:\Repo\rocketseat.ci.api> 

```

## Associando o PVC na nossa aplicação

Persistent Volume Claim “reinvindicar” (verbo to claim, em inglês)

Para utilizar um PVC em um pod ou em um deployment, ele deve ser montado como um volume. 

Exemplo:

StorageClass => Valume : Persistent - 10Gi => Clain - pvc - 1Gi


![](image/Kubernetes/diagram-PV-PVC-pod.webp)



Neste exemplo, o pod monta o PVC `my-pvc` no caminho `/usr/share/nginx/html`.

## Resumo sobre volumes

![](image/Kubernetes/diagram-PV-PVC-pod.webp)


- **Volumes** permitem o armazenamento persistente de dados em Kubernetes, mesmo após o ciclo de vida dos pods.
- **StorageClass** define as políticas de provisionamento dinâmico de armazenamento.
- **PersistentVolume (PV)** representa a unidade de armazenamento física ou virtual configurada.
- **PersistentVolumeClaim (PVC)** é a solicitação de armazenamento feita por um pod.
- O uso de PVCs em aplicações é essencial para manter a persistência e facilitar o gerenciamento de dados em clusters Kubernetes.

