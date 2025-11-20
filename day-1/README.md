# Descomplicando o Kubernetes

## DAY-1

## O que vimos hoje?

Durante o Day-1 nós vamos entender o que é um container, vamos falar sobre a importância do container runtime e do container engine. Durante o Day-1 vamos entender o que é o Kubernetes e sua arquitetura, vamos falar sobre o control plane, workers, apiserver, scheduler, controller e muito mais! Será aqui que iremos criar o nosso primeiro cluster Kubernetes e realizar o deploy de um pod do Nginx. O Day-1 é para que eu possa me sentir mais confortável com o Kubernetes e seus conceitos iniciais.

---

## Inicio da aula do Day-1

### Qual a distro GNU/Linux que devo usar?

Devido ao fato de algumas ferramentas importantes, como o systemd e journald, terem se tornado padrão na maioria das principais distribuições disponíveis hoje, você não deve encontrar problemas para seguir o treinamento, caso você opte por uma delas, como Ubuntu, Debian, CentOS e afins.

### Alguns sites que devemos visitar

Abaixo temos os sites oficiais do projeto do Kubernetes:

- https://kubernetes.io
- https://github.com/kubernetes/kubernetes/
- https://github.com/kubernetes/kubernetes/issues

---

## O Container Engine

Antes de começar a falar um pouco mais sobre o Kubernetes, nós primeiro precisamos entender alguns componentes que são importantes no ecossistema do Kubernetes, um desses componentes é o Container Engine.

O Container Engine é o responsável por gerenciar as imagens e volumes, é ele o responsável por garantir que os os recursos que os containers estão utilizando está devidamente isolados, a vida do container, storage, rede, etc.

Hoje temos diversas opções para se utilizar como Container Engine, que até pouco tempo atrás tinhamos somente o Docker para esse papel.

Opções como o Docker, o CRI-O e o Podman são bem conhecidas e preparadas para o ambiente produtivo. O Docker, como todos sabem, é o Container Engine mais popular e ele utiliza como Container Runtime o containerd.

Container Runtime? O que é isso?

Calma que vou te explicar já já, mas antes temos que falar sobre a OCI. :)

---

## OCI - Open Container Initiative

A OCI é uma organização sem fins lucrativos que tem como objetivo padronizar a criação de containers, para que possam ser executados em qualquer ambiente. A OCI foi fundada em 2015 pela Docker, CoreOS, Google, IBM, Microsoft, Red Hat e VMware e hoje faz parte da Linux Foundation.

O principal projeto criado pela OCI é o runc, que é o principal container runtime de baixo nível, e utilizado por diferentes Container Engines, como o Docker. O runc é um projeto open source, escrito em Go e seu código está disponível no GitHub.

Agora sim já podemos falar sobre o que é o Container Runtime.

---

## O Container Runtime

Para que seja possível executar os containers nos nós é necessário ter um Container Runtime instalado em cada um dos nós.

O Container Runtime é o responsável por executar os containers nos nós. Quando você está utilizando Docker ou Podman para executar containers em sua máquina, por exemplo, você está fazendo uso de algum Container Runtime, ou melhor, o seu Container Engine está fazendo uso de algum Container Runtime.

Temos três tipos de Container Runtime:

- **Low-level**: são os Container Runtime que são executados diretamente pelo Kernel, como o runc, o crun e o runsc.

- **High-level**: são os Container Runtime que são executados por um Container Engine, como o containerd, o CRI-O e o Podman.

- **Sandbox**: são os Container Runtime que são executados por um Container Engine e que são responsáveis por executar containers de forma segura em unikernels ou utilizando algum proxy para fazer a comunicação com o Kernel. O gVisor é um exemplo de Container Runtime do tipo Sandbox.

- **Virtualized**: são os Container Runtime que são executados por um Container Engine e que são responsáveis por executar containers de forma segura em máquinas virtuais. A performance aqui é um pouco menor do que quando temos um sendo executado nativamente. O Kata Containers é um exemplo de Container Runtime do tipo Virtualized.

---

## O que é o Kubernetes?

**Versão resumida:**

O projeto Kubernetes foi desenvolvido pela Google, em meados de 2014, para atuar como um orquestrador de contêineres para a empresa. O Kubernetes (k8s), cujo termo em Grego significa "timoneiro", é um projeto open source que conta com design e desenvolvimento baseados no projeto Borg, que também é da Google. Alguns outros produtos disponíveis no mercado, tais como o Apache Mesos e o Cloud Foundry, também surgiram a partir do projeto Borg.

Como Kubernetes é uma palavra difícil de se pronunciar - e de se escrever - a comunidade simplesmente o apelidou de k8s, seguindo o padrão i18n (a letra "k" seguida por oito letras e o "s" no final), pronunciando-se simplesmente "kates".

## Arquitetura do k8s

- **API Server**: É um dos principais componentes do k8s. Este componente fornece uma API que utiliza JSON sobre HTTP para comunicação, onde para isto é utilizado principalmente o utilitário kubectl, por parte dos administradores, para a comunicação com os demais nós. Estas comunicações entre componentes são estabelecidas através de requisições REST.

- **etcd**: O etcd é um datastore chave-valor distribuído que o k8s utiliza para armazenar as especificações, status e configurações do cluster. Todos os dados armazenados dentro do etcd são manipulados apenas através da API. Por questões de segurança, o etcd é por padrão executado apenas em nós classificados como control plane no cluster k8s, mas também podem ser executados em clusters externos, específicos para o etcd, por exemplo.

- **Scheduler**: O scheduler é responsável por selecionar o nó que irá hospedar um determinado pod (a menor unidade de um cluster k8s - não se preocupe sobre isso por enquanto, nós falaremos mais sobre isso mais tarde) para ser executado. Esta seleção é feita baseando-se na quantidade de recursos disponíveis em cada nó, como também no estado de cada um dos nós do cluster, garantindo assim que os recursos sejam bem distribuídos. Além disso, a seleção dos nós, na qual um ou mais pods serão executados, também pode levar em consideração políticas definidas pelo usuário, tais como afinidade, localização dos dados a serem lidos pelas aplicações, etc.

- **Controller Manager**: É o controller manager quem garante que o cluster esteja no último estado definido no etcd. Por exemplo: se no etcd um deploy está configurado para possuir dez réplicas de um pod, é o controller manager quem irá verificar se o estado atual do cluster corresponde a este estado e, em caso negativo, procurará conciliar ambos.

- **Kubelet**: O kubelet pode ser visto como o agente do k8s que é executado nos nós workers. Em cada nó worker deverá existir um agente Kubelet em execução. O Kubelet é responsável por de fato gerenciar os pods que foram direcionados pelo controller do cluster, dentro dos nós, de forma que para isto o Kubelet pode iniciar, parar e manter os contêineres e os pods em funcionamento de acordo com o instruído pelo controlador do cluster.

- **Kube-proxy**: Age como um proxy e um load balancer. Este componente é responsável por efetuar roteamento de requisições para os pods corretos, como também por cuidar da parte de rede do nó.

### Portas que devemos nos preocupar

**CONTROL PLANE**

| Protocol | Direction | Port Range  | Purpose                    |
|----------|-----------|-------------|----------------------------|
| TCP      | Inbound   | 6443*       | Kubernetes API server      |
| TCP      | Inbound   | 2379-2380   | etcd server client API     |
| TCP      | Inbound   | 10250       | Kubelet API                |
| TCP      | Inbound   | 10251       | kube-scheduler             |
| TCP      | Inbound   | 10252       | kube-controller-manager    |

**WORKERS**

| Protocol | Direction | Port Range   | Purpose      |
|----------|-----------|--------------|--------------|
| TCP      | Inbound   | 10250        | Kubelet API  |
| TCP      | Inbound   | 30000-32767  | NodePort     |

### Conceitos-chave do k8s

- **Pod**: É o menor objeto do k8s. Como dito anteriormente, o k8s não trabalha com os contêineres diretamente, mas organiza-os dentro de pods, que são abstrações que dividem os mesmos recursos, como endereços, volumes, ciclos de CPU e memória. Um pod pode possuir vários contêineres.

- **Deployment**: É um dos principais controllers utilizados. O Deployment, em conjunto com o ReplicaSet, garante que determinado número de réplicas de um pod esteja em execução nos nós workers do cluster. Além disso, o Deployment também é responsável por gerenciar o ciclo de vida das aplicações, onde características associadas a aplicação, tais como imagem, porta, volumes e variáveis de ambiente, podem ser especificados em arquivos do tipo yaml ou json para posteriormente serem passados como parâmetro para o kubectl executar o deployment. Esta ação pode ser executada tanto para criação quanto para atualização e remoção do deployment.

- **ReplicaSets**: É um objeto responsável por garantir a quantidade de pods em execução no nó.

- **Services**: É uma forma de você expor a comunicação através de um ClusterIP, NodePort ou LoadBalancer para distribuir as requisições entre os diversos Pods daquele Deployment. Funciona como um balanceador de carga.

---


## Criando um cluster Kubernetes

### Kind

O Kind (Kubernetes in Docker) é outra alternativa para executar o Kubernetes num ambiente local para testes e aprendizado, mas não é recomendado para uso em produção.


### Criando um cluster com o Kind

Após realizar a instalação do Kind, vamos iniciar o nosso cluster.

```bash
kind create cluster


```

É possível criar mais de um cluster e personalizar o seu nome.

```bash
kind create cluster --name descomplicando


kubectl cluster-info --context kind-descomplicando

```

Para visualizar os seus clusters utilizando o kind, execute o comando a seguir.

```bash
kind get clusters
```

Liste os nodes do cluster.

```bash
kubectl get nodes
```

### Criando um cluster com múltiplos nós locais com o Kind

É possível para essa aula incluir múltiplos nós na estrutura do Kind, que foi mencionado anteriormente.

Execute o comando a seguir para selecionar e remover todos os clusters locais criados no Kind.

```bash
kind delete clusters descomplicando

```

Crie um arquivo de configuração para definir quantos e o tipo de nós no cluster que você deseja. No exemplo a seguir, será criado o arquivo de configuração kind-3nodes.yaml para especificar um cluster com 1 nó control-plane (que executará o control plane) e 2 workers.

```bash
cat << EOF > $HOME/kind-cluster.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
EOF
```

Agora vamos criar um cluster chamado descomplicando utilizando as especificações definidas no arquivo kind-3nodes.yaml.

```bash
kind create cluster --name descomplicando --config kind-cluster.yaml


kubectl cluster-info --context kind-descomplicando

```

Valide a criação do cluster com o comando a seguir.

```bash
kubectl get nodes
```

Mais informações sobre o Kind estão disponíveis em: https://kind.sigs.k8s.io

---

## Primeiros passos no k8s

### Verificando os namespaces e pods

O k8s organiza tudo dentro de namespaces. Por meio deles, podem ser realizadas limitações de segurança e de recursos dentro do cluster, tais como pods, replication controllers e diversos outros. Para visualizar os namespaces disponíveis no cluster, digite:

```bash
kubectl get namespaces
```

Vamos listar os pods do namespace kube-system utilizando o comando a seguir.

```bash
kubectl get pod -n kube-system
```

Será que há algum pod escondido em algum namespace? É possível listar todos os pods de todos os namespaces com o comando a seguir.

```bash
kubectl get pods -A
```

Há a possibilidade ainda, de utilizar o comando com a opção `-o wide`, que disponibiliza maiores informações sobre o recurso, inclusive em qual nó o pod está sendo executado. Exemplo:

```bash
kubectl get pods -A -o wide
```

### Executando nosso primeiro pod no k8s

Iremos iniciar o nosso primeiro pod no k8s. Para isso, executaremos o comando a seguir.

```bash
kubectl run nginx --image nginx

pod/nginx created
```

Listando os pods com `kubectl get pods`, obteremos a seguinte saída.

```
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          66s
```

Vamos agora remover o nosso pod com o seguinte comando.

```bash
kubectl delete pod nginx
```

A saída deve ser algo como:

```
pod "nginx" deleted
```

Uma outra forma de criar um pod ou qualquer outro objeto no Kubernetes é através da utilização de um arquivo manifesto, que é um arquivo em formato YAML onde você passa todas as definições do seu objeto. Mas pra frente vamos falar muito mais sobre como construir arquivos manifesto, mas agora eu quero que você conheça a opção `--dry-run` do kubectl, pois com ele podemos simular a criação de um resource e ainda ter um manifesto criado automaticamente.

**Exemplos:**

Para a criação do template de um pod:

```bash
kubectl run pod-nginx --image nginx --dry-run=client -o yaml > pod.yaml
```

Aqui estamos utilizando ainda o parâmetro `-o`, utilizado para modificar a saída para o formato YAML.

Com o arquivo gerado em mãos, agora você consegue criar um pod utilizando o manifesto que criamos da seguinte forma:

```bash
kubectl apply -f pod.yaml
```

### Expondo o pod e criando um Service

Dispositivos fora do cluster, por padrão, não conseguem acessar os pods criados, como é comum em outros sistemas de contêineres. Para expor um pod, execute o comando a seguir.

```bash
kubectl expose pod pod-nginx
```

Será apresentada a seguinte mensagem de erro:

```
error: couldn't find port via --port flag or introspection
See 'kubectl expose -h' for help and examples
```

O erro ocorre devido ao fato do k8s não saber qual é a porta de destino do contêiner que deve ser exposta (no caso, a 80/TCP). Para configurá-la, vamos primeiramente remover o nosso pod antigo:

```bash
kubectl delete -f pod.yaml
```

Agora vamos executar novamente o comando para a criação do pod utilizando o parâmetro `dry-run`, porém agora vamos adicionar o parâmetro `--port` para dizer qual a porta que o container está escutando, lembrando que estamos utilizando o nginx nesse exemplo, um webserver que escuta por padrão na porta 80.

```bash
kubectl run pod-nginx --image nginx --port 80 --dry-run=client -o yaml > pod.yaml
kubectl create -f pod.yaml
```

Liste os pods.

```bash
kubectl get pods

NAME        READY   STATUS    RESTARTS   AGE
pod-nginx   1/1     Running   0          32s
```

O comando a seguir cria um objeto do k8s chamado de Service, que é utilizado justamente para expor pods para acesso externo.

```bash
kubectl expose pod pod-nginx
```

Podemos listar todos os services com o comando a seguir.

```bash
kubectl get services

NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP   8d
meu-nginx    ClusterIP   10.105.41.192   <none>        80/TCP    2m30s
```

Como é possível observar, há dois services no nosso cluster: o primeiro é para uso do próprio k8s, enquanto o segundo foi o que acabamos de criar.

### Limpando tudo e indo para casa

Para mostrar todos os recursos recém criados, pode-se utilizar uma das seguintes opções a seguir.

```bash
kubectl get all

kubectl get pod,service

kubectl get pod,svc
```

Note que o k8s nos disponibiliza algumas abreviações de seus recursos. Com o tempo você irá se familiarizar com elas. Para apagar os recursos criados, você pode executar os seguintes comandos.

```bash
kubectl delete -f pod-template.yaml
kubectl delete service meu-nginx
```

Liste novamente os recursos para ver se os mesmos ainda estão presentes.
