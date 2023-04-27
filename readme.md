# Kubernetes

Kubernetes é um orquestrador de containers capaz de resolver todos os problemas de vários containers rodando ao mesmo tempo.

- Kubernetes é capaz de criar e gerenciar um cluster para que nós consigamos manter a nossa aplicação estável
sempre que nós quisermos adicionar novos containers, sempre que nós quisermos reiniciar a nossa aplicação de maneira 
automática, caso ela tenha falhado. Então nós chamamnos isso de __orquestração de containers__

- Quando duas máquinas antinjam o seu limite de poder computacional em algum momento, o kubernetes também é capaz de criar uma nova máquina dentro de nosso cluster para que nós consigamos também adicionar mais container à essas máquinas e isso é claro, uma máquina virtual. Nós não vamos manipular uma máquina física aqui na nossa casa ou no nosso ambiente
de trabalho.

# A arquitetura do kubernetes

> Recursos
> - POD
> - RS
> - DEPLOY
> - VOL
> - HPA
> - PV
> - IMG
> - PVC
> - SVC
> - SC
> - DS 
> - QUOTA

PODS são mais do um recurso que encapsula um container no kubernete.Nós nunca vamos utilizar um container propriamente, nós vamos utilizar pods.

- Elas podem ser master, onde o master é responsável por coordenar e gerenciar todo o nosso luster e nós temos os nodes que são responsáveis pela execução do trabalho duro, para executar os nossos pods em capsulas containers, por assim dizer.


__master:__
- Gerenciar o cluster
- Manter e atualiza o estado
- Receber e executar novos comandos
    - control plane
        - API
        - C-M
        - SCHED
        - etcd

__node:__
- Executar as aplicações
  - ***Node***
    - kubelet
    - k-proxy

![](https://s.profissionaisti.com.br/wp-content/uploads/2018/07/kubernetes-arquitetura.jpg)

O comando kubectl ele:
- cria --> run
- ler --> get
- atualizar --> edit
- remove --> delete

De forma __declarativa ou imperativo__
![](https://cdn-icons-png.flaticon.com/128/8790/8790376.png)

### Entendendo a API

__Como a comunicação é feita com a API?__

- Utilizando uma ferramenta chamada kubectl

## Cluster no linux

> **Baixar o miniekube e kubectl**
> **Baixar o virtualbox**

**Comando para inicializar o cluster no virtualbox**

```shell 
minekube start --vm-driver=virtualbox
```

**Verificar a quantiade de nós e algumas informações de nó**

```shell
kubectl get nodes
```

## Entendendo o que são pods.
> **Pode conter 1 ou mais containers de maneira declarativa ou imperativa.**

Quando criamos um pod ele ganha um endereço ip e dentro do Pod nós temos total liberdade de fazer um mapeamento de portas

***OBS!!*** Não pode existir 2 containers dentro de pod com as mesma porta

- Em caso de um único container falhar nesse momento, o pod vai parar de funcionar, kubernets vai criar um novo pod, mas com o endereço ip diferente do antigo pod.
  
    - **Pods são efêmeros!**
    - **São substituidos a qualquer momento**

🔵 Caso exista 2 containers dentro de um pod, e um  dos containers pare de funcionar, o outro vai estar rodando normal, logo, esse pod ainda vai estar funcionando normalmente.

🔵 Se caso os 2 containers parem de funcionar, ai sim o pod é finalizado! Outro será criado em seu lugar!


- Compartilham namespaces de rede e ipc
- podem compartilhar volumes
  

**IPC -> Comunicação entre processos, é o grupo de macanismo que permite aos processos transferirem informações entre si.**

Dentro do pod, containers podem se comunicar via localhost com o mesmo ip do pod.

- Como possuem IPs diferentes, containers em pods diferentes podem utilizar o mesmo número de porta.
- containers dentro de mesmo pod conseguem de comunicar via localhost.

## Criando o primeiro POD de forma Imperativa

**Cria POD**
    
    kubectl run nginx-pod --image=nginx:latest

**Mostrar nossos PODs**

    kubectl get pods

**Mostrar nossos pods sendo executados em tempo real**

    kubectl get pods --watch

**Para descrever um POD**

    kubectl describe pod <nomePod>

**Para editar um POD**

    kubectl edit pod <pod>

## Para Saber mais: Onde as imagens sãp armazenadas.

Executamos o nosso primeiro Pod. Porém, como o Kubernetes armazena as imagens baixadas dentro do cluster?

A resposta é simples: quando definimos que um Pod será executado, o scheduler definirá em qual Node isso acontecerá. O resultado então é que as imagens quando baixadas de repositórios como o Docker Hub, serão armazenadas localmente em cada Node, não sendo compartilhada por padrão entre todos os membros do cluster.

## Criando Pods de maneira declarativa

- Crie um arquivo yaml

```yaml

#versao da api
apiVersion: v1
#informar o tipo de recurso
kind: Pod
#Definir informações
metadata:
  name: primeiro-pod-declarativo
#Especificações
spec:
  containers:
      #posso colocar qualquer nome
    - name: nginx-container
      #imagem
      image: nginx:latest
    #  - name: nginx-container
    #   #imagem
    #   image: nginx:latest
    #  - name: nginx-container
    #   #imagem
    #   image: nginx:latest

# OBS! - separa containers, posso colocar varios containers

```
'-' = Separa cada container; posso criar vários containers dentro de um pod

**Para aplicar/criar o pod, digite o seguinte comando**

> kubctl apply -f <arquivo.yaml>

**Deletando pods**

    kubectl delete pod nginx-pod

ou

    kubectl delete -f <arquivo>

**Para executar pod de maneira iterativa**

     kubectl exec -it <namePod> -- bash

### Quando um pod é dado como encerrado?

> **Quando todos os containers dentro do pod param de funcionar.**

## Explorando pods com services

### Conhecendo services

**Comando que mostra mais informação sobre o pod incluindo o ip**

    kubectl get pods -o wide

**Service(SVC)**

> - **Abstração para expor aplicações executando em um ou mais pods**
> - **Proveem IP's fixos para comunicação**
> - **Proveem um DNS para um ou mais pods**
> - **São capazes de fazer balanceamento de carga**

 - ClusterIp
 - NodePort
 - LoadBlance

### Vantagens de service

 - Fazem balanceamento de carga
 - Proveem IP's fixos para comunicação

**Comando para mostrar serviços**

    kubectl get svc

### Arquivo yaml de um SVC

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-pod-2
spec:
  type: ClusterIP
  selector:
    app: segundo-pod
  ports:
    - port: 80

```

**OBS! Ao criar uma label, ele vão receber uma chave e um valor**

> Podemos utilizar/criar várias labels


10.1.0.67:50 -> Ela não é estável

**Redireciona o ip do svc para o ip do pod**

- A porta pode variar

10.111.155.166:9000 --> 172.17.0.4:80

**Como um service sabe quais pods deve gerenciar?**

> Através de **labels** definidas no **metadata** e utilizando o campo **selector** no service.

**O que é um ClusterIP?**

Um serviço ClusterIP é o serviço padrão do Kubernetes. Ele fornece um serviço dentro do cluster que outros aplicativos dentro do cluster podem acessar. Não há acesso externo.

![](https://miro.medium.com/max/1400/1*I4j4xaaxsuchdvO66V3lAg.webp)

### Criando um Node Port

```yaml
apiVersion: v1
kind: Service
metadata:
  name: svc-pod-1
spec:
  type: NodePort
  ports:
    - port: 80
  selector:
    app: primeiro-pod
```

**O que é um NodePort?**

Um serviço NodePort é a maneira mais primitiva de obter tráfego externo diretamente para seu serviço. NodePort, como o nome indica, abre uma porta específica em todos os nós (as VMs), e qualquer tráfego enviado para essa porta é encaminhado para o serviço.

![](https://miro.medium.com/max/1400/1*CdyUtG-8CfGu2oFC5s0KwA.webp)

Basicamente, um serviço NodePort tem duas diferenças de um serviço “ClusterIP” normal. Primeiro, o tipo é “NodePort”. Há também uma porta adicional chamada nodePort que especifica qual porta abrir nos nós. Se você não especificar esta porta, ela escolherá uma porta aleatória. Na maioria das vezes, você deve deixar o Kubernetes escolher a porta; Como chocante diz, há muitas ressalvas sobre quais portas estão disponíveis para você usar.

**Quando usar?**

Existem muitas desvantagens nesse método:

1. Você só pode ter um serviço por porta
2. Você só pode usar as portas 30000–32767
3. Se o endereço IP do seu Node/VM ​​mudar, você precisa lidar com isso.

Por esses motivos, não recomendo usar esse método em produção para expor diretamente seu serviço. Se você estiver executando um serviço que não precisa estar sempre disponível ou for muito sensível aos custos, esse método funcionará para você. Um bom exemplo de tal aplicativo é um aplicativo de demonstração ou algo temporário.

**Mostrar o IP dos nó**

    kubectl get nodes -o wide

**obs! Para ter acesso a aplicação no kluster no linux é preciso acessar o internal-ip:nodePort, já no windows é so localhost:nodePort.**

```shell
NAME          TYPE        CLUSTER-IP      PORT(S)               
svc-1       NodePort     10.101.214.22   80:30000/TCP
```
- Dentro do cluster o service escuta na porta 80, enquanto fora do cluster escuta na porta 30000.
- Utilizamos o IP do nó para acessar o service através da porta 30000.

###  Criando um Load Balancer

LoadBalancer nada mais é do que um ClusterIP que permite a comunicação entre uma mna do mundo externo e os nosso pods. Só que ele automaticamente se integra ao LoadBalancer do nosso cloud provider.

- Utilizam automaticamente os balanceadores de carga de cloud providers.
- Por serem um Load Balancer, também são um NodePort e ClusterIP ao mesmo tempo.
- Um serviço LoadBalancer é a maneira padrão de expor um serviço à Internet. No GKE, isso ativará um balanceador de carga de rede que fornecerá um único endereço IP que encaminhará todo o tráfego para seu serviço.

![](https://miro.medium.com/max/1400/1*P-10bQg_1VheU9DRlvHBTQ.webp)

**Quando usar?**

Se você deseja expor diretamente um serviço, este é o método padrão. Todo o tráfego na porta que você especificar será encaminhado para o serviço. Não há filtragem, roteamento, etc. Isso significa que você pode enviar quase qualquer tipo de tráfego para ele, como HTTP, TCP, UDP, Websockets, gRPC ou qualquer outro.

## Resumo do módulo:

- O que são e para que servem os Services
- Como garantir estabilidade de IP e DNS
- Como criar um Service
- Labels são responsáveis por definir a relação Service x Pod
- Um ClusterIP funciona apenas dentro do cluster
- Um NodePort expõe Pods para dentro e fora do cluster
- Um LoadBalancer também é um NodePort e ClusterIP
- Um LoadBalancer é capaz de automaticamente utilizar um balanceador de carga de um cloud provider

## Aplicando services ao projeto

### Acessando o Portal

