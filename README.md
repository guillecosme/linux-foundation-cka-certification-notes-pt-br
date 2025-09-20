

# KCNA - Notas de Prpeparação da prova


 Preciso revisar os módulos:
  - IntroductioN
  - Core concepts
  - Scheduling

# Comandos úteis 

| Objetivo                                  | Comando          | Observações                                                                                                                                                                                                                          |
|-------------------------------------------|------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Editar as configs de um POD em execução   | kubectl edit pod | Nem sempre é possível editar as configs de um pod em execução, caso surja o erro de Forbidden, uma cópia do manifesto que você alterou será criada na pasta /tmp. Com o arquivo em mãoes você pode deletar o pod e criá-lo novamente |
| Deletar um POD em execução                | kvbectl delete pod <Nome do POD> | -                                                                                                                                                                                                                                    |
| Cria um POD com base num arquiv YAML      | kubectl create -f /Path do YAML/NOME DO YAML.yml | -                                                                                                                                                                                                                                    |
| Editar um DEPLOYMENT                      | kubectl edit deployment NOME_DEPLOYMENT | -                                                                                                                                                                                                                                    |
| Inspecionar os logs de execução de um pod | kubectl logs NOME_POD | -                                                                                                                                                                                                                                    |
| Verificar os dados de um POD              | kubectl describe POD_NAME | -                                                                                                                                                                                                                                    |
| Criar um DeamonSet a partir de um arquivo | kubectl create -f deamon-set-definition.yml | -                                                                                                                                                                                                                                    |
| Listar um DeamonSet                       | kubctl get deamonsets | -                                                                                                                                                                                                                                    |
| Verificar os detalhes de um DeamonSet | kubectl describe deamonsets NOME_DEAMONSET | -                                                                                                                                                                                                                                    |
| Verificar o yaml de um POD | kubectl get POD_NAME -o yaml | -                                                                                                                                                                                                                                    |
| Obter as informações de conexão de um node | kubectl get nodes -o wide | -                                                                                                                                                                                                                                    | 


## Scheduling & Workload Management
***

A capcidade de lançar pods em determinados nodes bseados em diferentes configurações


### Resource limits

Cpaz de fazer o processo de scheduling baseado em:

Quem comanda o processo é o kube-scheduler

- CPU
-  Memory

Isso pode ser feito no manifesto do pod através da sessão de "RESOURCES"

```yaml
resources:
  requests:
    memory: "4Gi"
    cpu: 2
```
é possível também configurar o máximo de memória que um Container de um pode pode utilizar a través da especificação  do do pod:


```yaml
resources:
  requests:
    memory: "4Gi"
      cpu: 2
  limits:
    memory: "4Gi"
    cpu: 2
```

Um container não pode usar mais CPU do que o definido no limite, caso ele tente utilizar CPU ele será deletado.
O contrário já não é uma verdade, quando se trata de memória o container pode ultrapassar o limite, mas ele tentará ser alocado em outro node com o erro OOM -> Out of Memory Error

Por default qualquer pod pode utilizar o máximo de memória e cpu que conseguir, só é repsieto limites se os limites forem declarados no manifesto do pod.

Requests =  Mínimo desejável
Limits = Máximo desejável

Vocẽ pode estar s eperguntando, mas se o default é não ter limits ou requests a não ser que você declare isso no manifesto, como então garantir que haja um comportamento padrão entre todos os pods?

Você utilizar o LImitRange no manifesto do seu Pod:

Especificação de CPU:

```yaml
apiVersion: v1
kind: LimitRange
metadata: cpu-resource-constraint
spec:
  limits:
    - default:
        cpu: 500m
      defaultRequest:
        cpu: 500m
      max:
        cpu: "1"
      min:
        cpu: 100m
      type: Container
```
Especificação de memória:

```yaml
apiVersion: v1
kind: LimitRange
metadata: cpu-resource-constraint
spec:
  limits:
  - default:
      cpu: 500m
    defaultRequest:
      memory: 1Gi
    max:
      memory: 1Gi
    min:
      memory: 500Mi
    type: Container
```

Também é possível estabelecer um quaota máxima de recurusos que os containers podem usar utilizando o Resources Quotas e aplicar no nível do NameSpace:

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
spec:
  hard:
    requests.cpu: 4
    request.memory: 4Gi
    limits.cpu: 10
    limits.memory: 10Gi
```

### DeamonSets

Assim como as ReplicasSets e Dployments o DeamonSet ajuda na escalabilidade e gerenciamento do ambeiente, ele basicamente garante que uma cópia de um determinado POD esteja sempre presente em todos os Nodes de um determinado namespace. Ele é muito útil em cenários de monitoring e agentes, onde você precisa de um agente de monitoria em todos os Nodes

Exemplos:
 - Calico
 - kube-proxy

Como declarar um DaemonSet:

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: monitoring-example
spec:
  selector:
    matchLabels:
      app: monitoring-example
  template:
    metadata:
      labels:
        app: monitoring-example
    spec:
      containers:
        - name: monitoring-example
          image: monitoring-example
```

### Static Pods

O kubelet é basicamente o capitão do Node, ele depende do Master Node  e seus componentes (kube-api, server, Control Manager, Scheduler e etcd) para coneguir gerenciar os pods do Node, porém se perdermos a função do master node o kubelet consegue gerenciar o node de maneira independente utilizando o Container Run Engine (containerD, Docker, rkt), você pode configurar o kubelet para ler as configurações de um pod em um servidor ou diretório externo sem dependencia do Master Node.
Esse mecanismo só pode ser feito com pods, não funciona com Replicasets, Deployments ou DaemonSets.

[Criando um Static Pod >> Doc Oficial](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)

Pra configurar você precisa alterar o arquivo de configuração do kublet no node, onde você aponta o novo caminho dos manifestos do pod direto no arquivo de configuração.

Quando um POD estático é criado ele é listado no comando kubectl get pods com pelo nome + sufixo  do nome do nome

Por padrão o manifesto para pods estáticos ficam no diretório /etc/kubernets/manifests. Porém para verificar é necessário checar o arquivo de configurações do kubet no node.

/var/libe/kubelet/config.yaml
