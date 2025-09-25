
# 📘 CKA - Resumo Completo de Preparação

Este documento contém uma revisão detalhada dos principais tópicos cobrados no exame **KCNA (Kubernetes and Cloud Native Associate)**, com explicações, exemplos, comandos úteis e links para a documentação oficial.

---

# Componentes

# 📘 CKA Study Guide – Component Maintenance

## 🔹 Introdução
Neste guia vamos detalhar **atualizações e manutenções** dos principais componentes de um cluster Kubernetes:
- kube-apiserver
- kube-controller-manager
- kube-scheduler
- kubelet
- kube-proxy
- kubectl
- etcd

Documentação oficial:
- [Cluster Components](https://kubernetes.io/docs/concepts/overview/components/)

---

## 🛠️ Componentes

### 1. kube-apiserver
- **Função:**
    - É o **ponto central de comunicação** do cluster.
    - Todos os comandos `kubectl` e interações de componentes passam pelo API Server.
    - Responsável por validar requisições, autenticar usuários e expor a API REST.

- **Manutenção:**
    - Monitorar logs e checar conectividade com `kubectl`.
    - Escalar réplicas em ambientes com HA.

- **Atualização:**
    - Em clusters `kubeadm`, o upgrade é feito com:
      ```bash
      kubeadm upgrade apply v1.30.0
      ```
    - O Pod do `kube-apiserver` é recriado no namespace `kube-system`.

- **Logs:**
  ```bash
  kubectl -n kube-system logs kube-apiserver-<node-name>
  ```

- **Documentação:**  
  [kube-apiserver](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-apiserver/)

---

### 2. kube-controller-manager
- **Função:**
    - Executa **control loops** que monitoram o estado do cluster e garantem que o estado atual se alinhe ao desejado.
    - Inclui controladores de replicação, endpoints, namespaces, etc.

- **Manutenção:**
    - Monitorar logs para identificar falhas em control loops.
    - Garantir alta disponibilidade em clusters HA.

- **Atualização:**
    - Feita com `kubeadm upgrade apply`, que recria o Pod do controller-manager.

- **Logs:**
  ```bash
  kubectl -n kube-system logs kube-controller-manager-<node-name>
  ```

- **Documentação:**  
  [kube-controller-manager](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-controller-manager/)

---

### 3. kube-scheduler
- **Função:**
    - Responsável por **agendar Pods** em nós disponíveis.
    - Leva em conta afinidades, tolerations, recursos e políticas de scheduling.

- **Manutenção:**
    - Monitorar logs para problemas de agendamento.
    - Validar eventos do cluster para identificar pods não agendados.

- **Atualização:**
    - Feita com `kubeadm upgrade apply`, que recria o Pod do scheduler.

- **Logs:**
  ```bash
  kubectl -n kube-system logs kube-scheduler-<node-name>
  ```

- **Documentação:**  
  [kube-scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/)

---

### 4. kubelet
- **Função:**
    - Agente que roda em cada nó.
    - Garante que os Pods definidos pelo API Server estejam rodando no nó.
    - Monitora containers via runtime (containerd, cri-o, docker).

- **Manutenção:**
    - Reiniciar o serviço se houver falhas:
      ```bash
      sudo systemctl restart kubelet
      ```

- **Atualização:**
    - Atualizado individualmente em cada nó:
      ```bash
      sudo apt-get install -y kubelet=1.30.0-00
      sudo systemctl daemon-reload
      sudo systemctl restart kubelet
      ```

- **Logs:**
  ```bash
  journalctl -u kubelet -f
  ```

- **Documentação:**  
  [kubelet](https://kubernetes.io/docs/reference/command-line-tools-reference/kubelet/)

---

### 5. kube-proxy
- **Função:**
    - Mantém as **regras de rede** em cada nó.
    - Implementa balanceamento de carga e roteamento para Services.
    - Pode rodar em modo `iptables` ou `ipvs`.

- **Manutenção:**
    - Validar se os Pods `kube-proxy` estão rodando como DaemonSet.
    - Analisar logs em caso de falhas de conectividade entre serviços.

- **Atualização:**
    - Feita atualizando a imagem do DaemonSet:
      ```bash
      kubectl -n kube-system set image ds/kube-proxy       kube-proxy=k8s.gcr.io/kube-proxy:v1.30.0
      ```

- **Logs:**
  ```bash
  kubectl -n kube-system logs -l k8s-app=kube-proxy
  ```

- **Documentação:**  
  [kube-proxy](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/)

---

### 6. kubectl
- **Função:**
    - Ferramenta de linha de comando para interagir com o cluster.
    - Usa a API do kube-apiserver.
    - Essencial para administração e troubleshooting.

- **Manutenção:**
    - Verificar compatibilidade de versão entre `kubectl` e servidor.

- **Atualização:**
    - Manual, no host do administrador:
      ```bash
      curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
      chmod +x kubectl && sudo mv kubectl /usr/local/bin/
      ```

- **Logs:**
    - Não gera logs próprios (client-side).

- **Documentação:**  
  [kubectl](https://kubernetes.io/docs/reference/kubectl/)

---

### 7. etcd
- **Função:**
    - Banco de dados chave/valor distribuído.
    - Armazena o **estado de todo o cluster** (informações de Pods, ConfigMaps, Secrets etc).
    - É o **componente mais crítico** do Kubernetes.

- **Manutenção:**
    - Realizar backups regulares:
      ```bash
      ETCDCTL_API=3 etcdctl snapshot save /var/lib/etcd/backup.db
      ```
    - Monitorar espaço em disco e latência.
    - Usar monitoramento e alertas para detectar corrupção ou lentidão.

- **Atualização:**
    - Controlada pelo `kubeadm upgrade apply`, que atualiza o Pod do etcd.
    - Em clusters externos, precisa ser atualizado manualmente:
      ```bash
      sudo apt-get install -y etcd=<version>
      ```

- **Restore:**
  ```bash
  ETCDCTL_API=3 etcdctl snapshot restore /var/lib/etcd/backup.db     --data-dir=/var/lib/etcd-from-backup
  ```

- **Logs:**
  ```bash
  kubectl -n kube-system logs etcd-<node-name>
  ```

- **Documentação:**  
  [Operating etcd](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)

---

## 📊 Tabela de Manutenção e Atualização por Componente

| Componente            | Função Principal                          | Logs/Verificação de Status                                        | Atualização                                        |
|------------------------|-------------------------------------------|-------------------------------------------------------------------|----------------------------------------------------|
| **kube-apiserver**     | Interface central (API REST)              | `kubectl -n kube-system logs kube-apiserver-<node>`               | `kubeadm upgrade apply`                            |
| **controller-manager** | Control loops                             | `kubectl -n kube-system logs kube-controller-manager-<node>`      | `kubeadm upgrade apply`                            |
| **kube-scheduler**     | Agenda Pods nos nós                       | `kubectl -n kube-system logs kube-scheduler-<node>`               | `kubeadm upgrade apply`                            |
| **kubelet**            | Agente nos nós                            | `journalctl -u kubelet -f`                                        | `apt-get install kubelet` + restart                |
| **kube-proxy**         | Regras de rede / Service routing          | `kubectl -n kube-system logs -l k8s-app=kube-proxy`               | Atualizado via DaemonSet (nova imagem)             |
| **kubectl**            | CLI de administração                      | `kubectl version --client --short`                                | Baixar binário mais recente                        |
| **etcd**               | Banco chave/valor do estado do cluster    | `kubectl -n kube-system logs etcd-<node>`                         | `kubeadm upgrade apply` ou atualização manual      |

---

## ✅ Resumo
- **etcd** é o componente mais crítico → sempre mantenha backups.
- **Planos de upgrade (`kubeadm upgrade plan`)** ajudam a atualizar control-plane com segurança.
- **kubelet** e **kubectl** são atualizados manualmente em cada nó/host.
- **kube-proxy** é atualizado trocando a imagem do DaemonSet.
- Monitorar logs de cada componente é essencial para troubleshooting.

---

## 1. 🔎 Core Concepts

### Pods
- Unidade mínima de execução no Kubernetes.  
- Pode conter **um ou mais containers** que compartilham:  
  - Rede (IP, portas)  
  - Volumes (armazenamento)  

**Comandos úteis:**
```bash
kubectl run nginx --image=nginx
kubectl get pods
kubectl describe pod <NOME>
```

👉 [Documentação oficial](https://kubernetes.io/docs/concepts/workloads/pods/)

---

### ReplicaSets
- Garante que o número desejado de **réplicas de Pods** esteja sempre em execução.  
- Substituído na prática por **Deployments**.  

**Exemplo YAML:**
```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: my-replicaset
spec:
  replicas: 3
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
        image: nginx
```

👉 [Documentação oficial](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/)

---

### Deployments
- Controla ReplicaSets.  
- Permite **rollouts** (atualizações) e **rollbacks**.  

**Comandos úteis:**
```bash
kubectl create deployment nginx --image=nginx
kubectl get deployments
kubectl rollout status deployment/nginx
kubectl rollout history deployment/nginx
kubectl rollout undo deployment/nginx
```

👉 [Documentação oficial](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/)

---

### Services
- Abstração de rede que expõe Pods.  
- Tipos principais:  
  - **ClusterIP** (interno, padrão)  
  - **NodePort** (exposição em portas do Node)  
  - **LoadBalancer** (integra com nuvem)  

**Comandos úteis:**
```bash
kubectl expose deployment nginx --port=80 --target-port=80 --type=ClusterIP
kubectl expose deployment nginx --type=NodePort --port=80
kubectl get svc
```

👉 [Documentação oficial](https://kubernetes.io/docs/concepts/services-networking/service/)

---

### Namespaces
- Isolam recursos dentro de um cluster.  
- Muito usados em multi-tenant clusters.  

**Comandos úteis:**
```bash
kubectl get namespaces
kubectl create namespace dev
kubectl get pods -n dev
```

👉 [Documentação oficial](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)

---

### ConfigMaps e Secrets
- **ConfigMaps**: armazenam configurações não sensíveis.  
- **Secrets**: armazenam informações sensíveis (base64).  

**Comandos úteis:**
```bash
kubectl create configmap app-config --from-literal=ENV=prod
kubectl get configmap app-config -o yaml

kubectl create secret generic db-secret --from-literal=PASSWORD=12345
kubectl get secret db-secret -o yaml
```

👉 [ConfigMaps](https://kubernetes.io/docs/concepts/configuration/configmap/)  
👉 [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)

---

## 2. ⚖️ Scheduling & Workload Management

### Requests e Limits
- **Requests**: recursos mínimos garantidos.  
- **Limits**: máximo permitido.  

**Exemplo YAML:**
```yaml
resources:
  requests:
    memory: "512Mi"
    cpu: "500m"
  limits:
    memory: "1Gi"
    cpu: "1"
```

👉 [Documentação oficial](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

---

### LimitRange
Define valores padrões e limites mínimos/máximos.  

**Exemplo:**
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-limit-range
spec:
  limits:
  - default:
      cpu: 500m
    defaultRequest:
      cpu: 200m
    max:
      cpu: "1"
    min:
      cpu: 100m
    type: Container
```

👉 [Documentação oficial](https://kubernetes.io/docs/concepts/policy/limit-range/)

---

### ResourceQuota
Limita os recursos **totais do namespace**.  

**Exemplo:**
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-resources
spec:
  hard:
    requests.cpu: "1"
    requests.memory: 1Gi
    limits.cpu: "2"
    limits.memory: 2Gi
```

👉 [Documentação oficial](https://kubernetes.io/docs/concepts/policy/resource-quotas/)

---

### DaemonSets
- Garante **1 Pod por Node**.  
- Usado para monitoramento, CNI, proxies.  

**Comandos úteis:**
```bash
kubectl get daemonsets
kubectl describe daemonset <NOME>
```

👉 [Documentação oficial](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/)

---

### Static Pods
- Gerenciados diretamente pelo **kubelet**.  
- Manifestos em `/etc/kubernetes/manifests/`.  

👉 [Documentação oficial](https://kubernetes.io/docs/tasks/configure-pod-container/static-pod/)

---

### PriorityClasses
- Define prioridade para Pods.  
- Pods de menor prioridade podem ser **preemptados**.  

**Exemplo:**
```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 100000
globalDefault: false
description: "High priority pods"
```

👉 [Documentação oficial](https://kubernetes.io/docs/concepts/scheduling-eviction/pod-priority-preemption/)

---

### Admission Controllers
- Validam ou modificam requests para o API Server.  
- Tipos:  
  - **ValidatingWebhook** (somente valida)  
  - **MutatingWebhook** (pode alterar objetos)  

👉 [Documentação oficial](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/)

---

## 3. 📊 Escalabilidade

### Horizontal Pod Autoscaler (HPA)
- Escala **horizontalmente** (réplicas de Pods).  
- Baseado em métricas (CPU, memória ou customizadas).  

**Exemplo:**
```bash
kubectl autoscale deployment nginx --cpu-percent=50 --min=1 --max=5
```

👉 [Documentação oficial](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)

---

### Vertical Pod Autoscaler (VPA)
- Ajusta automaticamente requests/limits de CPU e memória.  
- Melhor para workloads **estáveis**.  

👉 [Documentação oficial](https://github.com/kubernetes/autoscaler/tree/master/vertical-pod-autoscaler)

---

## 4. 🔎 Monitoring

### Metrics Server
- Agrega métricas de recursos de Pods e Nodes.  

**Comandos úteis:**
```bash
kubectl top nodes
kubectl top pods
```

👉 [Documentação oficial](https://github.com/kubernetes-sigs/metrics-server)

---

## 5. 🔄 Application Lifecycle Management

- **Rolling Update** (padrão) e **Recreate**.  
- **Rollback** possível com `kubectl rollout undo`.  
- **ConfigMaps & Secrets** permitem externalizar configs.  
- **Multi-Container Pods**:  
  - Init Containers  
  - Sidecars  

👉 [Documentação oficial](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#strategy)

---

## 6. Cluster Maintenece

### 6.1. Atualização de Clusters
- **Objetivo:** Garantir que os componentes do cluster estejam atualizados com versões suportadas.
- **Ferramentas comuns:**
    - `kubeadm upgrade`
    - Gerenciadores de cluster (GKE, EKS, AKS → upgrades automatizados).
- **Passos típicos em clusters `kubeadm`:**
    1. Planejar a atualização:
       ```bash
       kubeadm upgrade plan
       ```
    2. Atualizar o plano de controle:
       ```bash
       sudo kubeadm upgrade apply v1.30.0
       ```
    3. Atualizar o `kubelet` em cada nó:
       ```bash
       sudo apt-get install -y kubelet=1.30.0-00
       sudo systemctl restart kubelet
       ```
- Documentação: [Upgrading kubeadm clusters](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)

---

### 6.2. Manutenção de Nós
- **Cordoning (isolamento):** Evita que novos Pods sejam agendados em um nó.
  ```bash
  kubectl cordon <node-name>
  ```
- **Draining (remoção controlada):** Remove Pods em execução de forma segura.
  ```bash
  kubectl drain <node-name> --ignore-daemonsets --delete-emptydir-data
  ```
- **Uncordoning (reabilitar agendamento):**
  ```bash
  kubectl uncordon <node-name>
  ```

**Exemplo prático:**
```bash
# Preparar o nó para manutenção
kubectl cordon worker-1

# Mover os pods de forma segura
kubectl drain worker-1 --ignore-daemonsets

# Após manutenção, reabilitar
kubectl uncordon worker-1
```

---

### 6.3. Logs e Monitoramento de Componentes
- **Verificar estado dos componentes:**
  ```bash
  kubectl get componentstatuses
  ```
- **Logs do kubelet (no host):**
  ```bash
  journalctl -u kubelet -f
  ```
- **Logs de Pods de sistema (exemplo: etcd):**
  ```bash
  kubectl -n kube-system logs etcd-master
  ```

Documentação: [Troubleshooting Clusters](https://kubernetes.io/docs/tasks/debug/)

---

### 6.4. Backup e Restore do etcd
- O **etcd** é o banco de dados chave/valor que armazena o estado do cluster.
- Fazer backup regular é crítico para recuperação de desastre.

**Backup do etcd:**
```bash
ETCDCTL_API=3 etcdctl   --endpoints=https://127.0.0.1:2379   --cacert=/etc/kubernetes/pki/etcd/ca.crt   --cert=/etc/kubernetes/pki/etcd/server.crt   --key=/etc/kubernetes/pki/etcd/server.key   snapshot save /var/lib/etcd/backup.db
```

**Restore do etcd:**
```bash
ETCDCTL_API=3 etcdctl snapshot restore /var/lib/etcd/backup.db   --data-dir=/var/lib/etcd-from-backup
```

Documentação: [Operating etcd](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)

---

### 6.5. Troubleshooting Comum
- **Nó NotReady:**
    - Checar `kubelet` logs.
    - Validar comunicação com API Server.
- **Pods presos em Terminating:**
  ```bash
  kubectl delete pod <pod-name> --force --grace-period=0
  ```
- **Erro no etcd:**
    - Validar espaço em disco.
    - Restaurar de snapshot, se necessário.

---

## 📊 Tabela de Comandos Úteis de Manutenção

| Tarefa                                          | Comando                                                                 |
|-------------------------------------------------|-------------------------------------------------------------------------|
| Planejar upgrade kubeadm                        | `kubeadm upgrade plan`                                                  |
| Aplicar upgrade do cluster                      | `kubeadm upgrade apply vX.Y.Z`                                          |
| Cordon de nó  (Deixar o Node fora do scheduler) | `kubectl cordon <node>`                                                 |
| Drain de nó                                     | `kubectl drain <node> --ignore-daemonsets --delete-emptydir-data`       |
| Uncordon de nó                                  | `kubectl uncordon <node>`                                               |
| Verificar componentes                           | `kubectl get componentstatuses`                                         |
| Logs do kubelet                                 | `journalctl -u kubelet -f`                                              |
| Backup etcd                                     | `etcdctl snapshot save backup.db`                                       |
| Restore etcd                                    | `etcdctl snapshot restore backup.db --data-dir=/var/lib/etcd-from-backup` |
| Verificar estado dos nós                        | `kubectl get nodes -o wide`                                             |
| Checar pods em kube-system                      | `kubectl get pods -n kube-system -o wide`                               |

---

## ✅ Resumo
- Mantenha sempre **backups atualizados do etcd**.
- Use `cordon/drain/uncordon` para manutenções seguras.
- Sempre **planeje upgrades** (`kubeadm upgrade plan`).
- Domine **logs e troubleshooting** de componentes críticos (`kubelet`, `etcd`, `apiserver`).

---


##  🛠️ Tabela de Comandos Úteis

| Objetivo                               | Comando                                         |
|----------------------------------------|-------------------------------------------------|
| Criar recurso a partir de YAML         | `kubectl create -f arquivo.yaml`                |
| Editar Pod                             | `kubectl edit pod <NOME>`                       |
| Deletar Pod                            | `kubectl delete pod <NOME>`                     |
| Criar Deployment                       | `kubectl create deployment nginx --image=nginx` |
| Editar Deployment                      | `kubectl edit deployment <NOME>`                |
| Escalar Deployment                     | `kubectl scale deployment <NOME> --replicas=3`  |
| Atualizar imagem em Deployment         | `kubectl set image deployment/nginx nginx=img:v2` |
| Ver status de rollout                  | `kubectl rollout status deployment/nginx`       |
| Rollback de rollout                    | `kubectl rollout undo deployment/nginx`         |
| Logs de Pod                            | `kubectl logs <POD>`                            |
| Logs de Pod (multi-container)          | `kubectl logs <POD> -c <CONTAINER>`             |
| Métricas de Pods                       | `kubectl top pods`                              |
| Métricas de Nodes                      | `kubectl top nodes`                             |
| Expor serviço                          | `kubectl expose deployment nginx --port=80`     |

---

## 7. Como fazer a mnutenção do Cluster

## 🔹 Introdução
Um upgrade de cluster Kubernetes deve ser feito de forma **planejada e ordenada**, garantindo:
- Alta disponibilidade.
- Compatibilidade entre componentes.
- Backups do etcd antes de qualquer ação.

Este guia traz um **fluxo passo a passo** para atualizar **control-plane, kubelets, kube-proxy, kubectl e etcd**.

Documentação oficial:
- [Kubeadm upgrade guide](https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/)
- [Operating etcd](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)

---

## 🛠️ Checklist Passo a Passo

### 7.1. Pré-requisitos
- Backup completo do **etcd**:
  ```bash
  ETCDCTL_API=3 etcdctl snapshot save /var/lib/etcd/backup-$(date +%Y%m%d).db
  ```
- Verificar versões suportadas:
  ```bash
  kubectl version --short
  kubeadm version
  ```
- Checar nós e Pods do sistema:
  ```bash
  kubectl get nodes
  kubectl get pods -n kube-system -o wide
  ```

---

### 7.2. Planejar o Upgrade
```bash
kubeadm upgrade plan
```
- Mostra versão atual, versão alvo e componentes a serem atualizados.

---

### 7.3. Atualizar o Control Plane
1. Atualizar `kubeadm`:
   ```bash
   sudo apt-get install -y kubeadm=1.30.0-00
   ```
2. Aplicar upgrade:
   ```bash
   sudo kubeadm upgrade apply v1.30.0
   ```
3. Atualizar `kubelet` e `kubectl` no nó do control-plane:
   ```bash
   sudo apt-get install -y kubelet=1.30.0-00 kubectl=1.30.0-00
   sudo systemctl daemon-reload
   sudo systemctl restart kubelet
   ```

---

### 7.4. Atualizar os Nós Workers
1. Atualizar `kubeadm` no nó worker:
   ```bash
   sudo apt-get install -y kubeadm=1.30.0-00
   ```
2. Colocar o nó em modo manutenção:
   ```bash
   kubectl drain <worker-node> --ignore-daemonsets --delete-emptydir-data
   ```
3. Atualizar o nó worker:
   ```bash
   sudo kubeadm upgrade node
   sudo apt-get install -y kubelet=1.30.0-00
   sudo systemctl daemon-reload
   sudo systemctl restart kubelet
   ```
4. Reabilitar o nó:
   ```bash
   kubectl uncordon <worker-node>
   ```

---

### 7.5. Atualizar o kube-proxy
- Atualizar imagem do DaemonSet:
  ```bash
  kubectl -n kube-system set image ds/kube-proxy     kube-proxy=k8s.gcr.io/kube-proxy:v1.30.0
  ```

---

### 7.6. Atualizar o kubectl (admin hosts)
- Atualizar binário no host do administrador:
  ```bash
  curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
  chmod +x kubectl && sudo mv kubectl /usr/local/bin/
  ```

---

### 7.7. Atualizar o etcd
- Em clusters gerenciados por `kubeadm`, o `etcd` é atualizado junto ao control-plane.
- Para clusters externos (standalone etcd):
  ```bash
  sudo apt-get install -y etcd=<version>
  ```
- Validar integridade após upgrade:
  ```bash
  ETCDCTL_API=3 etcdctl endpoint health     --endpoints=https://127.0.0.1:2379     --cacert=/etc/kubernetes/pki/etcd/ca.crt     --cert=/etc/kubernetes/pki/etcd/server.crt     --key=/etc/kubernetes/pki/etcd/server.key
  ```

---

### 7.8. Pós-Upgrade (Validação)
- Validar versão do cluster:
  ```bash
  kubectl version --short
  ```
- Verificar componentes:
  ```bash
  kubectl get nodes
  kubectl get pods -n kube-system
  ```
- Verificar etcd:
  ```bash
  kubectl -n kube-system logs etcd-<node-name>
  ```

---

## 📊 Resumo do Fluxo
1. **Backup etcd.**
2. Planejar upgrade com `kubeadm upgrade plan`.
3. Atualizar control-plane (`kubeadm`, `kubelet`, `kubectl`).
4. Atualizar workers (drain → upgrade → uncordon).
5. Atualizar `kube-proxy`.
6. Atualizar `kubectl` nos hosts admin.
7. Atualizar `etcd`.
8. Validar cluster e serviços.

---

## 8 Segurança em clusters Kubernetes

# 🔒 CKA Study Guide – Segurança no Kubernetes

## 📘 Introdução
Segurança em Kubernetes é um dos pilares mais importantes do exame **CKA**. Um administrador precisa entender como proteger o cluster contra acessos indevidos, aplicar controle de permissões, usar certificados, proteger imagens, isolar workloads com políticas de rede e manter práticas seguras no uso de containers.  
Este guia cobre os principais tópicos de segurança exigidos no exame, com explicações detalhadas, links para documentação oficial e exemplos práticos de comandos.

---

## 8.1. 🔑 Autenticação e Autorização
No Kubernetes, segurança começa com **quem pode acessar** o cluster (**Autenticação**) e **o que pode ser feito** (**Autorização**).  
A autenticação pode ocorrer via certificados x509, tokens de ServiceAccounts, ou integração com provedores externos como OIDC. Após autenticado, o request passa por autorização, onde mecanismos como **RBAC** ou **Webhook Authorizers** decidem se a ação é permitida.  
É fundamental garantir que usuários humanos usem **credenciais individuais** e workloads usem **ServiceAccounts dedicadas**. Boas práticas incluem o princípio de **least privilege** (mínimo de permissões possíveis).

📖 Doc: [Authentication](https://kubernetes.io/docs/reference/access-authn-authz/authentication/) | [Authorization](https://kubernetes.io/docs/reference/access-authn-authz/authorization/)

**Exemplo:** listar usuários configurados no kubeconfig:
```bash
kubectl config view --minify -o jsonpath='{.users[*].name}'
```

---

## 8.2. 🛡️ Kubernetes Security Primitives
Kubernetes traz primitivas de segurança embutidas que permitem isolar workloads e controlar acessos. Entre elas estão:
- **Namespaces** para segmentação lógica.
- **RBAC** para gerenciar permissões de usuários e aplicações.
- **Network Policies** para restringir tráfego entre Pods.
- **ServiceAccounts** que dão identidade para workloads.  
  O uso combinado dessas primitivas fornece uma base sólida para segmentar ambientes, aplicar controles granulares e limitar o impacto em caso de comprometimento.

📖 Doc: [Security Primitives](https://kubernetes.io/docs/concepts/security/)

**Exemplo:** criar namespace isolado para workloads seguros:
```bash
kubectl create namespace secure-apps
```

---

## 8.3. 📜 TLS no Kubernetes
### TLS Completo (Simétrica, Assimétrica, CAs, Certificados e Handshake)

### 📚 Referências Primárias (RFCs e Especificações)
- **TLS 1.3:** RFC 8446 — *The Transport Layer Security (TLS) Protocol Version 1.3*
- **TLS 1.2:** RFC 5246 — *The Transport Layer Security (TLS) Protocol Version 1.2*
- **X.509 Certs:** RFC 5280 — *Internet X.509 Public Key Infrastructure Certificate and CRL Profile*
- **CSR (PKCS #10):** RFC 2986 — *Certification Request Syntax Specification (PKCS #10)*
- **Hostname Verification:** RFC 6125 — *Service Identity in TLS*

---


### Conceitos de Criptografia

### 🔑 Criptografia Simétrica
- **Uma única chave secreta** para cifrar e decifrar.
- **Muito rápida** (usada para proteger o tráfego de dados após o handshake).
- Exemplos: **AES-GCM**, **ChaCha20-Poly1305** (ambos AEAD: autenticam e cifram).

```text
[SIMÉTRICA]
Chave Secreta (K)
   |-- Encrypt -->  Ciphertext = ENC_K(plaintext)
   |-- Decrypt -->  Plaintext  = DEC_K(ciphertext)
```

### 🔐 Criptografia Assimétrica
- **Par de chaves**: pública (divulgada) e privada (segredo).
- Mais **lenta**, usada principalmente para **autenticação** e **troca de chaves**.
- Exemplos: **RSA**, **ECDSA** (assinatura), **(EC)DHE** (acordo de chaves efêmeras).

```text
[ASSIMÉTRICA]
Chave Pública do Servidor  -> usada pelo cliente p/ verificar assinatura ou cifrar um segredo
Chave Privada do Servidor  -> usada pelo servidor p/ assinar ou decifrar o segredo
```

### 🧪 MAC, AEAD e Integridade
- TLS moderno usa **AEAD** (ex.: AES-GCM, ChaCha20-Poly1305) que provê **confidencialidade + integridade** no mesmo modo.
- Em TLS 1.2, também existiam ciphers **MAC-then-Encrypt** (HMAC separado). Em 1.3, apenas **AEAD**.

---

### PKI: Certificados X.509 e CAs

### 🏛️ Cadeia de Confiança
- **Root CA** (autoconfiada) → **Intermediate CA** → **Certificado do Servidor**.
- Navegadores/SOs confiam nas *Root CAs* pré-instaladas e validam a **cadeia**.

```text
+------------------+        +---------------------+        +--------------------------+
| Root CA (self-signed) |->  | Intermediate CA(s) |  ->    | Server Certificate (X.509)|
+------------------+        +---------------------+        +--------------------------+
             Assina                   Assina                       Apresentado no TLS
```

### 📄 Certificado X.509 (visão geral)
- **Sujeito (Subject)**: identidade (ex.: CN=api.example.com)
- **Emissor (Issuer)**: CA que assinou
- **Validade**: NotBefore / NotAfter
- **Chave Pública** e **Algoritmo de Assinatura**
- **Extensões importantes**:
    - **SAN (Subject Alternative Name)** — nomes DNS/IP válidos
    - **Key Usage / Extended Key Usage** — ex.: TLS Server Auth, Client Auth

### 🧾 CSR (PKCS #10)
- Solicitação de certificado contendo a **chave pública** e **provas** de posse da chave privada (assinatura da CSR).

---

### Objetivos do TLS
- **Confidencialidade**: ninguém lê o tráfego.
- **Integridade**: tráfego não é alterado sem detecção.
- **Autenticação**: cliente confia que fala com o servidor certo (e opcionalmente vice‑versa).
- **Forward Secrecy (PFS)**: segredos de sessão passados não são revelados mesmo que a chave privada vaze no futuro (via **(EC)DHE** efêmero).

---

### Handshake TLS 1.2 (detalhado)

> TLS 1.2 permite RSA estático para troca de chaves (**sem PFS**) e (EC)DHE (**com PFS**). Aqui focamos no caso **ECDHE** (recomendado).

### 🧭 Fluxo de Mensagens (ECDHE)
```text
Cliente                                                Servidor
   |-- ClientHello (versão, ciphers, curvas, random_C) --->|
   |<-- ServerHello (cipher escolhido, random_S) -----------|
   |<-- Certificate (cadeia do servidor) -------------------|
   |<-- ServerKeyExchange (ECDHE params, assinatura) -------|
   |<-- ServerHelloDone ------------------------------------|
   |-- ClientKeyExchange (ECDHE pub do cliente) ----------->|
   |-- ChangeCipherSpec ----------------------------------->|
   |-- Finished (protegido com chaves derivadas) ---------->|
   |<-- ChangeCipherSpec -----------------------------------|
   |<-- Finished (protegido) -------------------------------|
         [CANAL SIMÉTRICO ATIVO COM AEAD/MAC]
```

### 🔧 Como as chaves nascem (1.2 com ECDHE)
1. **ECDHE**: cliente e servidor trocam **pontos públicos**.
2. Cada lado calcula o **segredo compartilhado** `Z = ECDH(priv_local, pub_remoto)`.
3. Deriva‑se o **pre_master_secret** a partir de `Z`.
4. Usa‑se a PRF de TLS 1.2 (HMAC) com `random_C` e `random_S` para gerar o **master_secret**.
5. Do `master_secret` → deriva‑se o **key_block** (chaves simétricas + IVs).

```text
ECDHE: Z ---------------> pre_master_secret
pre_master_secret + randoms --PRF--> master_secret
master_secret --PRF--> key_block -> { client_write_key, server_write_key, IVs, ... }
```

### 🧩 Papel da Assimétrica vs. Simétrica (1.2)
- **Assimétrica**:
    - O **certificado do servidor** contém a **chave pública**; o servidor prova posse da privativa ao **assinar** os parâmetros ECDHE (*ServerKeyExchange*).
    - Opcional: **certificado do cliente** (mTLS).
- **Simétrica**:
    - Após derivar `key_block`, todo tráfego usa **AES-GCM** ou similar (**rápido** e eficiente).

---

## Handshake TLS 1.3 (detalhado)

> Simplifica o protocolo, **apenas AEAD**, **apenas (EC)DHE**, **PFS obrigatória**, **1-RTT** (um ida‑volta) e otimizações de reuso de sessão.

### 🧭 Fluxo de Mensagens (1-RTT)
```text
Cliente                                                Servidor
   |-- ClientHello + KeyShare(C_ECDHE) -------------------->|
   |<-- ServerHello + KeyShare(S_ECDHE) --------------------|
   |<-- EncryptedExtensions --------------------------------|
   |<-- Certificate (opcional p/ server auth) --------------|
   |<-- CertificateVerify ----------------------------------|
   |<-- Finished -------------------------------------------|
   |-- Finished ------------------------------------------->|
        [TRÁFEGO APLICACIONAL PROTEGIDO (AEAD)]
```

### 🌲 Árvore de Segredos (HKDF em 1.3)
- TLS 1.3 usa **HKDF** para derivação: **Early Secret**, **Handshake Secret**, **Master Secret**.
- Acordo ECDHE produz `Z` → mistura nos segredos com **salts** e **labels** padronizados → gera **traffic secrets**.

```text
          PSK? -> Early Secret
                 |
ECDHE Z ----> Handshake Secret
                 |
             Master Secret
                 |
    +------------+------------+
    |                         |
Handshake Traffic      Application Traffic
Secrets (c/s)          Secrets (c/s)
```

- **Finished** é calculado com chaves derivadas (garante integridade do handshake).
- **KeyUpdate** permite rotacionar chaves de aplicação periodicamente.

### 🧩 Assimétrica vs. Simétrica (1.3)
- **Assimétrica**:
    - Autenticação do servidor via **Certificate** + **CertificateVerify** (assinatura com chave **privada** do server).
    - Opcionalmente o cliente também apresenta certificado (mTLS).
- **Simétrica**:
    - Todo tráfego pós‑handshake cifrado via **AEAD** derivado da **Master Secret** (rápido e seguro).

---

### mTLS (Mutual TLS)
- Além de o cliente validar o **servidor**, o **servidor também valida o cliente**.
- Requer **CA** confiável para os **certificados de cliente** e validação de **Key Usage / EKU** apropriados.

```text
Cliente (cert + chave priv.)  <----TLS---->  Servidor (cert + chave priv.)
          \_________ autenticação mútua _______/
```

---

### Resumption e 0-RTT
- **Resumption**: reuso de segredos (via PSK/Tickets) para conexões subsequentes mais rápidas.
- **0-RTT (TLS 1.3)**: permite enviar dados de aplicativo no *primeiro voo* (antes do Finished), **sujeito a replay**.
    - Use apenas para idempotentes e avalie riscos de repetição.

---

### Comparação TLS 1.2 vs TLS 1.3

| Característica             | TLS 1.2                                   | TLS 1.3                                    |
|---------------------------|-------------------------------------------|--------------------------------------------|
| Handshake                 | 2-RTT típico                              | 1-RTT                                      |
| Troca de chaves           | RSA, DHE, ECDHE (PFS opcional)            | Apenas (EC)DHE (PFS sempre)                 |
| Criptografia              | AEAD e também MAC-then-Encrypt            | Apenas AEAD (ex.: AES-GCM, ChaCha20-Poly1305)|
| Segurança padrão          | Algoritmos fracos ainda existem           | Algoritmos fracos removidos                 |
| Resumption                | Session IDs/Tickets                       | PSK/Tickets + 0‑RTT                         |
| Simplicidade              | Mais complexa                             | Mais simples                                |

---

### Exemplos Práticos com OpenSSL

### 🔎 Inspecionar um certificado remoto
```bash
openssl s_client -connect api.example.com:443 -servername api.example.com </dev/null 2>/dev/null | openssl x509 -noout -text
```

### 🔗 Verificar cadeia e nome do host (RFC 6125)
```bash
openssl s_client -connect api.example.com:443 -servername api.example.com </dev/null \
  | openssl x509 -noout -subject -issuer -dates
openssl s_client -connect api.example.com:443 -servername api.example.com -verify_hostname api.example.com
```

### 🧾 Gerar par de chaves e CSR (PKCS#10)
```bash
# Chave privada (ECC P-256)
openssl ecparam -name prime256v1 -genkey -noout -out server.key
# CSR com SAN via config
cat > csr.conf <<'EOF'
[ req ]
prompt = no
distinguished_name = dn
req_extensions = req_ext
[ dn ]
CN = api.example.com
O = Example Org
C = BR
[ req_ext ]
subjectAltName = @alt_names
keyUsage = digitalSignature, keyEncipherment
extendedKeyUsage = serverAuth
[ alt_names ]
DNS.1 = api.example.com
DNS.2 = api.internal
EOF
openssl req -new -key server.key -out server.csr -config csr.conf
```

### 🔐 Forçar versões/ciphers em teste
```bash
# TLS 1.3
openssl s_client -tls1_3 -ciphersuites TLS_AES_128_GCM_SHA256 -connect api.example.com:443
# TLS 1.2
openssl s_client -tls1_2 -cipher 'ECDHE+AESGCM' -connect api.example.com:443
```

---

### TLS no Kubernetes

### 🔗 Onde TLS aparece no K8s
- **kube-apiserver**: apresenta **certificado de servidor** (SAN inclui hostname/IP do API server).
- **kubelet**: autentica‑se ao API usando **certificado de cliente** (mTLS).
- **etcd**: normalmente roda com **certificados de peer** e **server cert** separados (SANs adequados).
- **kube-controller-manager/kube-scheduler**: falam com o API server usando **client certs**.
- **kube-proxy**: também se autentica no API server (client cert/token).

```text
   [kubectl] --(TLS/mTLS)--> [kube-apiserver] --(mTLS)--> [etcd]
                 ^                     ^                         ^
                 |                     |                         |
            client cert           server cert/SAN           peer/server certs
```

### 🧩 Dicas de prova (erros comuns)
- **SAN faltando** no cert do apiserver (ex.: IP do cluster/hostname): conexões falham por nome/IP.
- **Key Usage/EKU** incorretos (serverAuth vs clientAuth).
- **Cert expirado** ou **CA errada**.
- **etcd** com **peer cert** incorreto → cluster quebra.

### 📂 Caminhos (kubeadm padrão)
```text
/etc/kubernetes/pki/ca.crt
/etc/kubernetes/pki/apiserver.crt
/etc/kubernetes/pki/apiserver.key
/etc/kubernetes/pki/apiserver-kubelet-client.crt
/etc/kubernetes/pki/apiserver-kubelet-client.key
/etc/kubernetes/pki/etcd/ca.crt
/etc/kubernetes/pki/etcd/server.crt
/etc/kubernetes/pki/etcd/server.key
/etc/kubernetes/pki/etcd/peer.crt
/etc/kubernetes/pki/etcd/peer.key
/etc/kubernetes/pki/etcd/healthcheck-client.crt
/etc/kubernetes/pki/etcd/healthcheck-client.key
```

---

## Tabela de Comandos Úteis

| Objetivo | Comando |
|---|---|
| Ver cadeias/versões do servidor | `openssl s_client -connect host:443 -servername host` |
| Mostrar certificado em texto | `... | openssl x509 -noout -text` |
| Verificar SAN/Validade | `openssl x509 -in cert.crt -noout -text` |
| Verificar host (RFC 6125) | `openssl s_client ... -verify_hostname host` |
| Listar ciphers local | `openssl ciphers -v` |
| Testar TLS 1.3 | `openssl s_client -tls1_3 -connect host:443` |
| Testar TLS 1.2 | `openssl s_client -tls1_2 -connect host:443` |
| Gerar key (EC P-256) | `openssl ecparam -name prime256v1 -genkey -noout -out server.key` |
| Criar CSR com SAN | `openssl req -new -key server.key -out server.csr -config csr.conf` |

---

## 🔎 Visão de Alto Nível: Como TLS usa Simétrica + Assimétrica para assegurar a comunicação
1. **Assimétrica autentica**: o servidor **prova identidade** assinando dados (1.2/1.3) com sua **chave privada**; o cliente valida com a **chave pública** do certificado (cadeia X.509 verificada até uma CA confiável).
2. **(EC)DHE cria segredo efêmero**: cliente e servidor trocam valores **públicos** e cada um calcula o **mesmo segredo compartilhado** `Z` sem transmiti‑lo (Diffie‑Hellman).
3. **HKDF/PRF deriva chaves**: a partir de `Z` e aleatórios do handshake, ambos derivam **segredos de tráfego** (sem nunca enviar a chave simétrica pela rede).
4. **Simétrica protege dados**: com chaves derivadas, o tráfego é cifrado via **AEAD** (rápido) garantindo confidencialidade + integridade.
5. **PFS**: como as chaves são efêmeras, mesmo que a chave privada do servidor vaze, **sessões antigas permanecem seguras**.

### Diagrama ASCII Integrado (Linha do Tempo)
```text
Tempo --->

[1] Autenticação (Assimétrica)             [2] Acordo de Chaves (ECDHE)           [3] Tráfego (Simétrica)
Cliente -----------------------------> Servidor
   |  Recebe Cert (chave pública)         |  Gera par ECDHE (e assina params)
   |  Verifica cadeia (CA)                |  Envia pub_ECDHE + assinatura
   |  Confia no nome (SAN/Hostname)       |
   |-- envia pub_ECDHE ------------------>|  Calcula Z = ECDH(priv_S, pub_C)
   |  Calcula Z = ECDH(priv_C, pub_S)     |
   |== HKDF/PRF(master/traffic keys) =====|  (mesmo Z, mesmas chaves derivadas)
   |<======== AEAD (dados do app) =======>|  Confidencialidade + Integridade
```

---

### ✅ Conclusão
- TLS combina **criptografia assimétrica** (autenticação + acordo) e **simétrica** (proteção eficiente do tráfego) para atingir **confidencialidade, integridade, autenticação e PFS**.
- **TLS 1.3** simplifica e fortalece o protocolo, devendo ser a **preferência** quando possível.
- Para o **CKA**, entenda **cadeias X.509, SANs, EKU/KeyUsage, caminhos de certificados no kubeadm, e como depurar conexões** (openssl).

Boa prática: **automatize a rotação** de certificados (apiserver/kubelet/etcd) e **monitore validade** para evitar indisponibilidades.

---

## 8.4. ⚙️ KubeConfig
O arquivo `kubeconfig` define como usuários e ferramentas se conectam ao cluster. Ele contém informações como **clusters, usuários, contextos e credenciais**.  
Manter esse arquivo seguro é fundamental, pois quem tiver acesso pode operar o cluster com as permissões configuradas. Recomenda-se configurar múltiplos contextos para separar ambientes (dev, stage, prod).  
Além disso, boas práticas incluem limitar o tempo de validade dos tokens e evitar que chaves privadas fiquem expostas em sistemas de versionamento.

📖 Doc: [Organizing Cluster Access Using kubeconfig Files](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)

**Exemplo:** trocar contexto ativo para ambiente de produção:
```bash
kubectl config use-context prod-admin
```

---

## 8.5. 📚 API Groups
A API do Kubernetes é dividida em **grupos** que organizam recursos de forma lógica. O grupo **Core (v1)** contém recursos básicos como Pods e Services, enquanto grupos como `apps/v1` e `rbac.authorization.k8s.io/v1` abrangem funcionalidades mais avançadas.  
Entender os grupos de API é essencial para criar regras de RBAC e YAMLs corretos. Além disso, conhecer as versões disponíveis ajuda a evitar recursos obsoletos.  
Para o exame CKA, é importante saber diferenciar recursos Core de recursos que pertencem a grupos específicos.

📖 Doc: [Kubernetes API Overview](https://kubernetes.io/docs/reference/using-api/)

**Exemplo:** listar todas as versões de API disponíveis:
```bash
kubectl api-versions
```

---

## 8.6. 👤 RBAC Roles
Roles permitem aplicar permissões dentro de um namespace específico. Elas definem quais operações podem ser realizadas em determinados recursos, aplicando granularidade e segurança.  
Uma Role é sempre combinada com um **RoleBinding**, que associa a Role a usuários, grupos ou ServiceAccounts. Esse modelo garante que workloads só consigam executar operações estritamente necessárias, reduzindo riscos.  
Sempre aplique o princípio de “menor privilégio” ao criar Roles.

📖 Doc: [Using RBAC Authorization](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

**Exemplo:** Role para leitura de Pods em um namespace:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: secure-apps
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```

---

## 8.7. 🌐 ClusterRoles
Enquanto as Roles se limitam a namespaces, as **ClusterRoles** permitem aplicar permissões em nível de cluster.  
Elas são úteis para recursos que não pertencem a nenhum namespace (como Nodes, PersistentVolumes) ou para fornecer permissões comuns a múltiplos namespaces.  
Assim como Roles, devem ser usadas em conjunto com um **ClusterRoleBinding**. Um erro comum é conceder permissões amplas demais em ClusterRoles, aumentando a superfície de ataque.

📖 Doc: [ClusterRole and ClusterRoleBinding](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)

**Exemplo:** ClusterRole com acesso de leitura a todos os Nodes:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list", "watch"]
```

---

## 8.8. 🧑‍💻 Service Accounts
ServiceAccounts fornecem identidade para Pods se autenticarem na API.  
Cada Pod por padrão recebe uma conta de serviço no namespace default, mas é uma boa prática criar ServiceAccounts dedicadas para cada aplicação.  
Isso permite aplicar RBAC específico para workloads, evitando que aplicações tenham mais acesso que o necessário. Tokens de ServiceAccounts podem ser montados como volumes ou injetados automaticamente em Pods.

📖 Doc: [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

**Exemplo:** criar uma ServiceAccount:
```bash
kubectl create serviceaccount my-sa -n secure-apps
```

---

## 8.9. 🖼️ Image Security
Manter segurança em imagens de containers é essencial. Use sempre imagens de registries confiáveis, mantenha versões atualizadas e verifique assinaturas quando disponíveis.  
Além disso, configure `imagePullSecrets` para acessar registries privados com segurança. Ferramentas como Trivy podem ser usadas para escanear imagens em busca de vulnerabilidades.  
Evite usar a tag `latest`, pois isso dificulta auditorias e pode causar inconsistência entre ambientes.

📖 Doc: [Image Pull Secrets](https://kubernetes.io/docs/concepts/containers/images/)

**Exemplo:** criar secret para autenticação em registry privado:
```bash
kubectl create secret docker-registry regcred   --docker-server=myregistry.io   --docker-username=user   --docker-password=pass   --docker-email=user@example.com
```

---

## 8.10. 🐳 Security nos Docker Containers
Além do Kubernetes, containers precisam de segurança no nível do Docker. Boas práticas incluem:
- Rodar containers como **non-root**.
- Evitar `allowPrivilegeEscalation`.
- Usar `readOnlyRootFilesystem` sempre que possível.
- Definir `PodSecurityContext` para grupos e usuários corretos.  
  Essas práticas reduzem a possibilidade de exploração de vulnerabilidades dentro do container.

📖 Doc: [Pod Security Standards](https://kubernetes.io/docs/concepts/security/pod-security-standards/)

**Exemplo:** rodar Pod como usuário não-root:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: app
    image: nginx
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
```

---

## 8.11. 🌐 Network Policies
As Network Policies permitem restringir tráfego de entrada e saída entre Pods. Sem elas, todo tráfego é permitido por padrão.  
Com Network Policies, é possível implementar um modelo de **zero trust**, onde somente comunicações explicitamente permitidas ocorrem.  
Elas são aplicadas por namespace e podem controlar tanto tráfego **Ingress** quanto **Egress**. Sua efetividade depende do CNI (Container Network Interface) usado no cluster.

📖 Doc: [Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)

**Exemplo:** política que permite apenas tráfego HTTP do frontend para o backend:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
  namespace: secure-apps
spec:
  podSelector:
    matchLabels:
      role: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 80
```

---

## 8.12. 🔄 Kubectx e Kubens
Gerenciar múltiplos clusters e namespaces pode ser confuso. As ferramentas **kubectx** e **kubens** ajudam a alternar rapidamente entre contextos e namespaces.  
Isso reduz erros operacionais, como aplicar comandos em clusters errados. Embora não façam parte do core do Kubernetes, são ferramentas muito úteis no dia a dia de administradores.  
No exame CKA, podem ajudar a ganhar tempo ao alternar ambientes durante os exercícios práticos.

📖 Repositório oficial: [ahmetb/kubectx](https://github.com/ahmetb/kubectx)

**Exemplos:**
```bash
# Trocar rapidamente de cluster
kubectx my-cluster

# Trocar de namespace
kubens secure-apps
```

---

## 📊 Tabela de Comandos Úteis de Segurança

| Tarefa                                 | Comando                                                                 |
|----------------------------------------|-------------------------------------------------------------------------|
| Listar usuários configurados           | `kubectl config view --minify -o jsonpath='{.users[*].name}'`           |
| Criar namespace seguro                 | `kubectl create namespace secure-apps`                                  |
| Ver certificados do API Server         | `openssl s_client -connect 127.0.0.1:6443 -showcerts`                   |
| Listar versões de API disponíveis      | `kubectl api-versions`                                                  |
| Criar Service Account                  | `kubectl create serviceaccount my-sa -n secure-apps`                     |
| Criar Secret de registry privado       | `kubectl create secret docker-registry regcred ...`                     |
| Trocar contexto                        | `kubectl config use-context my-context`                                 |
| Trocar cluster com kubectx             | `kubectx my-cluster`                                                    |
| Trocar namespace com kubens            | `kubens secure-apps`                                                    |
| Verificar pods no namespace seguro     | `kubectl get pods -n secure-apps`                                       |

---

## ✅ Resumo
- Entenda profundamente **autenticação e autorização** no Kubernetes.
- Domine **RBAC, Roles e ClusterRoles** para controle de permissões.
- Configure e renove **certificados TLS** corretamente.
- Use **ServiceAccounts** para workloads com permissões mínimas.
- Garanta segurança em **imagens, pods e containers**.
- Implemente **Network Policies** para isolar workloads.
- Use ferramentas como **kubectx/kubens** para facilitar a administração.

---



## 🗂️ 9. Storage (Introdução de Storage no Linux)

No Linux, o **storage** é organizado em um sistema de arquivos hierárquico, onde tudo é tratado como arquivo ou diretório, incluindo dispositivos de armazenamento. Os dispositivos são montados em pontos de montagem (mount points) para ficarem acessíveis no sistema. O kernel gerencia drivers de dispositivos de bloco (block devices), que são usados para armazenar dados de maneira persistente. Partições podem ser criadas para organizar discos, e sistemas de arquivos como **ext4, XFS** são comuns em ambientes Linux. É importante entender permissões, inodes e links simbólicos para administração de volumes. Além disso, conceitos como **LVM (Logical Volume Manager)** permitem flexibilidade para redimensionamento e gerenciamento de discos. Em containers, o sistema de arquivos base do host é frequentemente usado como camada de persistência.

**Exemplo:**
```bash
# Listar dispositivos de bloco
lsblk

# Montar manualmente um disco
mount /dev/sdb1 /mnt/data

# Ver espaço em disco
df -h
```

**Referências:**
- [Filesystem Hierarchy Standard](https://refspecs.linuxfoundation.org/FHS_3.0/fhs/index.html)
- [Linux Storage Administration](https://tldp.org/HOWTO/LVM-HOWTO/)

---

## 9.1 🐳 Storage no Docker

O Docker utiliza um sistema de **layers** para armazenar imagens e contêineres. Por padrão, o armazenamento é feito em `/var/lib/docker`. Cada contêiner possui um sistema de arquivos baseado em camadas copy-on-write (CoW). Para persistir dados além do ciclo de vida do contêiner, o Docker permite o uso de **volumes** e **bind mounts**. Volumes são gerenciados pelo Docker e ficam armazenados no host, enquanto bind mounts mapeiam diretórios do host diretamente no contêiner. Volumes são mais recomendados para ambientes de produção, pois são mais portáveis e independentes da estrutura de diretórios do host.

**Exemplo:**
```bash
# Criar volume
docker volume create meu_volume

# Rodar contêiner com volume
docker run -d -v meu_volume:/app/data nginx

# Ver detalhes dos volumes
docker volume inspect meu_volume
```

**Referências:**
- [Docker Storage Overview](https://docs.docker.com/storage/)
- [Docker Volumes](https://docs.docker.com/storage/volumes/)

---

## 9.2 🔌 Volume Driver Plugin no Docker

O Docker suporta **plugins de volume** para conectar contêineres a sistemas de armazenamento externos como NFS, Ceph, EBS, ou outros provedores de cloud. Drivers podem ser instalados para habilitar funcionalidades avançadas como snapshot, replicação ou criptografia. Ao criar volumes com drivers específicos, é possível integrar com sistemas de storage corporativos ou cloud providers. É importante passar opções de configuração no momento da criação do volume para customizar o comportamento do driver.

**Exemplo:**
```bash
# Criar volume usando driver nfs
docker volume create   --driver local   --opt type=nfs   --opt o=addr=192.168.1.100,rw   --opt device=:/path/to/dir   meu_nfs_volume
```

**Referências:**
- [Docker Volume Plugins](https://docs.docker.com/engine/extend/legacy_plugins/)

---

## 9.3 🧩 Container Storage Interface (CSI)

O **Container Storage Interface (CSI)** é um padrão que permite que qualquer provedor de armazenamento crie plugins para Kubernetes de forma padronizada. Antes do CSI, existiam drivers embutidos no Kubernetes, o que dificultava a manutenção e evolução. Com o CSI, o Kubernetes se torna independente do ciclo de desenvolvimento de cada provedor de storage. Drivers CSI são instalados como DaemonSets e controlam a criação, montagem e desmontagem de volumes. Isso permite suporte a soluções como EBS, GCP PD, Ceph, entre outros.

**Exemplo Manifesto CSI (trecho):**
```yaml
apiVersion: storage.k8s.io/v1
kind: CSIDriver
metadata:
  name: ebs.csi.aws.com
spec:
  attachRequired: true
```

**Referências:**
- [Kubernetes CSI Docs](https://kubernetes.io/docs/concepts/storage/volumes/#csi)
- [CSI Spec](https://github.com/container-storage-interface/spec)

---

## 📦 Volumes

Volumes no Kubernetes são recursos que permitem persistir dados além do ciclo de vida de um pod. Eles são montados nos contêineres e podem ser de tipos diferentes: `emptyDir` (temporário), `hostPath` (diretório do host), `nfs`, `configMap`, `secret`, `persistentVolumeClaim` entre outros. A principal vantagem é que os volumes sobrevivem a reinicializações de contêineres dentro do mesmo pod.

**Exemplo:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-com-volume
spec:
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - name: meu-volume
          mountPath: /usr/share/nginx/html
  volumes:
    - name: meu-volume
      emptyDir: {}
```

**Referências:**
- [Kubernetes Volumes](https://kubernetes.io/docs/concepts/storage/volumes/)

---

## 9.4 🗄️ Persistent Volumes (PV)

Um **Persistent Volume (PV)** é um recurso do cluster que representa um pedaço de armazenamento físico ou lógico provisionado. PVs são independentes do ciclo de vida de pods e são definidos pelo administrador ou provisionados dinamicamente. Podem usar diversos backends como NFS, iSCSI, AWS EBS, Azure Disk, etc. PVs têm um ciclo de vida próprio e podem ter políticas de `Retain`, `Delete` ou `Recycle`. Eles são fundamentais para ambientes produtivos onde dados não podem ser perdidos.

**Exemplo PV:**
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-nfs
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  nfs:
    path: /exports/data
    server: 192.168.1.100
  persistentVolumeReclaimPolicy: Retain
```

**Referências:**
- [Persistent Volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/)

---

## 9.5 📜 Persistent Volume Claims (PVC)

Um **Persistent Volume Claim (PVC)** é uma solicitação feita por um usuário para consumir armazenamento de um PV. Ele especifica o tamanho e o modo de acesso desejado. O Kubernetes tenta ligar (bind) o PVC a um PV compatível. A diferença chave é que **PV é o recurso físico ou lógico**, enquanto **PVC é o pedido de uso**. Se não houver PV adequado, o PVC fica em estado "Pending" até que um volume compatível seja disponibilizado.

**Exemplo PVC:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-nfs
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 5Gi
```

**Referências:**
- [Persistent Volume Claims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#persistentvolumeclaims)

---

## 9.6 📂 Using PVCs in Pods

Para usar um PVC em um Pod, basta referenciá-lo dentro da seção `volumes`. Assim, o Kubernetes garante que o volume solicitado será montado no contêiner. Isso desacopla o pod do detalhe do backend de armazenamento, permitindo maior portabilidade e abstração.

**Exemplo Pod usando PVC:**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-com-pvc
spec:
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - name: dados
          mountPath: /usr/share/nginx/html
  volumes:
    - name: dados
      persistentVolumeClaim:
        claimName: pvc-nfs
```

**Referências:**
- [Using Persistent Volume Claims](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes)

---

## 9.7 🏷️ Storage Class

Um **StorageClass** define **como volumes são provisionados dinamicamente** no Kubernetes. Ele especifica o provisionador (driver CSI ou in-tree), parâmetros de configuração e política de `reclaim`. Cada PVC pode pedir uma StorageClass específica para provisionamento automático de PVs. Sem uma StorageClass definida, os PVCs precisam de PVs pré-criados manualmente.

**Exemplo StorageClass:**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: sc-rapido
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
```

---



## 10. Networking no Docker
- O Docker cria por padrão a **bridge `docker0`**, conectando containers a uma sub-rede privada com **NAT** para saída via host.
- **Redes bridge definidas pelo usuário** oferecem isolamento e DNS embutido entre containers na mesma rede (melhor do que a bridge padrão).
- **`-p/--publish`** mapeia portas do container para o host (ex.: `-p 8080:80`), diferente do modelo do Kubernetes onde o **Pod tem IP roteável no cluster**.
- Modos **`host`** (container compartilha a stack de rede do host) e **`none`** (sem interface) existem, mas têm pouco uso em produção moderna com orquestradores.
- Em contextos de orquestração (Swarm), redes **overlay (VXLAN)** conectam nós distintos; no K8s, isso é delegado a um **CNI plugin**, não ao Docker.
- O isolamento se dá via **namespaces** e **veth pairs**; o tráfego de saída usa **iptables (MASQUERADE)** para NAT.
- O DNS entre containers numa bridge usa o **embedded DNS** do Docker; no K8s, o DNS é provido por **CoreDNS**.
- **Papel no K8s**: Kubernetes não usa “Docker networking” diretamente; o kubelet invoca **CNI** para configurar a rede do **Pod**.
- Cuidado: publicar portas via Docker **não equivale** a expor workloads no K8s (lá usamos **Service/Ingress/Gateway**).
- Diagnóstico: `docker network inspect`, `ip addr`, `iptables -t nat -S` ajudam a entender fluxo e NAT do host.
- Para estudo: compare **bridge+NAT** (Docker) vs **IP por Pod e roteamento** (K8s/CNI).
- Em ambientes modernos, **container runtime ≠ plano de rede**: o runtime executa containers, o **CNI configura rede**.

**🔧 Exemplos rápidos (paralelos no K8s):**
```bash
# Docker – redes
docker network ls
docker network inspect bridge
docker run --rm --network bridge -p 8080:80 nginx:alpine

# Kubernetes – "equivalente" conceitual: Pod com hostNetwork
kubectl run hostnet --image=nginx:alpine --overrides='{"spec":{"hostNetwork":true}}'
kubectl get pod -o wide hostnet
```

> **⚖️ Diferenças-chave**: Docker publica portas no host; **K8s** usa **Service (ClusterIP/NodePort/LoadBalancer)** e **Ingress/Gateway** para entrada. DNS no Docker vem do engine; no K8s, **CoreDNS** e **Service discovery**.

**📚 Referências:**
- Kubernetes – [Networking (Visão Geral)](https://kubernetes.io/docs/concepts/services-networking/)
- Kubernetes – [Container Runtimes](https://kubernetes.io/docs/setup/production-environment/container-runtimes/)
- Docker – [Networking Overview](https://docs.docker.com/network/)

---

## 10.1 Pod Networking (Modelo IP-per-Pod)
- **Cada Pod recebe seu próprio IP** e todas as containers do Pod compartilham **o mesmo namespace de rede** (lo, interfaces, portas).
- Premissa do K8s: **todos os Pods podem se comunicar sem NAT** (pod-to-pod) e com **qualquer Node** do cluster.
- A implementação concreta desse roteamento vem do **CNI plugin** (Calico, Cilium, Flannel, etc.) – Kubernetes define o **modelo**, não a tecnologia.
- **Pod IP é efêmero**: reinícios/realocação podem mudar IP; por isso, a comunicação estável usa **Services**.
- **kube-proxy** programa regras (iptables/ipvs) para **Service VIPs** apontarem para **Endpoints/EndpointSlices** dos Pods.
- **Node IP ≠ Pod IP**: tráfego pod↔externo pode passar por NAT/masquerade, conforme política do CNI e rota de saída.
- **hostNetwork: true** faz o Pod compartilhar a rede do Node (ganha performance, perde isolamento/porta única por host).
- **NetworkPolicy** controla tráfego L3/L4 entre Pods/Namespaces (necessita CNI compatível); por padrão, tudo é **permitido**.
- **Service types** (ClusterIP/NodePort/LoadBalancer) resolvem exposição interna/externa; **Ingress/Gateway** lida com L7 (HTTP).
- Em **dual-stack**, Pod/Service podem ter **IPv4+IPv6**; requer configuração do cluster e do CNI.
- Diagnóstico frequente: `kubectl exec` para `ping`, `curl`, `dig` dentro de Pods de teste.
- Performance/latência dependem do modo (overlay, eBPF, roteamento BGP, ipvs, etc.) provido pelo CNI.

**🔧 Exemplos (comandos úteis):**
```bash
kubectl get pods -o wide -A
kubectl run netshoot --image=nicolaka/netshoot -it --rm -- bash
# Dentro do netshoot:
ip addr && ip route && ss -lntp
# Testes:
curl -I http://my-svc.my-ns.svc.cluster.local
```

**⚖️ Diferenças importantes:**
- **Pod vs Container**: Pod é a **unidade de rede/implantação**; containers dentro dele compartilham IP/portas.
- **Pod IP vs Service IP**: Pod IP muda; Service IP é **estável** para descoberta e balanceamento.

**📚 Referências:**
- Kubernetes – [Cluster Networking](https://kubernetes.io/docs/concepts/cluster-administration/networking/)
- Kubernetes – [Pods](https://kubernetes.io/docs/concepts/workloads/pods/)
- Kubernetes – [Services](https://kubernetes.io/docs/concepts/services-networking/service/)

---

## 10.2 CNI no Kubernetes
- **CNI (Container Network Interface)** é uma especificação/contrato: o kubelet chama **plugins** para **ADD/DEL** interfaces ao ciclo de vida do Pod.
- Os **binários** dos plugins ficam em `/opt/cni/bin` e as **configs** em `/etc/cni/net.d/` (ordem lexicográfica define qual usar).
- Plugins populares: **Calico** (BGP/roteado), **Cilium** (eBPF), **Flannel** (VXLAN), **Weave Net**; há também **Multus** (múltiplas interfaces).
- Fluxo típico: criar **veth pair**, anexar ao namespace do Pod, configurar **IPAM**, rotas, e políticas (quando suportadas).
- **CNI chaining** permite encadear plugins (ex.: uma interface principal + policy plugin).
- **Kubenet** é legado e limitado; **use um CNI completo** para NetworkPolicy e recursos avançados.
- O **kubelet** não implementa rede; ele **invoca** o CNI e falha a criação do Pod se a configuração for inválida.
- Log e troubleshooting: ver logs do CNI (systemd, `/var/log/…`), checar **Node routes**, `ip a`, `ip r`, `tc`, `bpftool` (Cilium).
- Escolha de CNI impacta **encapsulamento, latência, MTU, segurança (policy), observabilidade** e integração com **Cloud Routes/BGP**.
- Em **managed K8s**, alguns CNIs são **gerenciados** (EKS, GKE, AKS); valide **limitações** por provedor.
- Plugins de **policy** (Calico, Cilium) implementam NetworkPolicy L3/L4; Cilium adiciona **L7** com eBPF.
- **IPAM** costuma vir integrado ao CNI (host-local, cluster-pools, etc.), mas pode ser externo.

**🔧 Comandos/manifests:**
```bash
# Ver diretórios CNI (Node)
ls -l /opt/cni/bin
ls -l /etc/cni/net.d

# Exemplo: instalar CNI (genérico – ajuste ao seu cluster)
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

**⚖️ Diferenças-chave:**
- **CNI ≠ CNM (Docker)**: K8s segue **CNI**; Docker Swarm segue **CNM**/libnetwork.
- **Kubenet vs CNI completo**: kubenet não suporta NetworkPolicy; CNI completo, sim.

**📚 Referências:**
- Kubernetes – [CNI (Plugins de Rede)](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/)
- Projeto CNI – [Spec](https://github.com/containernetworking/cni/blob/main/SPEC.md)

---

## 10.3 IPAM (IP Address Management) no K8s
- **IPAM** aloca endereços a Pods/Interfaces; pode ser **host-local** (por Node), **cluster pool** (global) ou **DHCP**, conforme o plugin.
- O **kube-controller-manager** pode anunciar **podCIDR** por Node (dependendo do modo/Cloud provider) para os CNIs consumirem.
- É crítico **planejar CIDRs**: **Pod CIDR** (ex.: 10.244.0.0/16) não deve colidir com redes do datacenter/VPC/VNet e **Service CIDR** (ex.: 10.96.0.0/12).
- **Service CIDR** define range para **ClusterIP**; não é roteável externamente e costuma trafegar via kube-proxy (iptables/ipvs).
- Em **dual-stack**, planeje **ranges IPv4 e IPv6** para Pods e Services; verifique se o CNI suporta.
- **Exaustão de IP** causa falha de criação de Pods; monitore consumo por **Namespace/Node** e aumente pools quando possível.
- Alguns CNIs (Calico/Cilium) suportam **pools múltiplos**, seleção por **labels**/namespaces, e reservas por **nodes**.
- **MTU** e encapsulamento (VXLAN/Geneve) impactam fragmentação; ajuste para evitar **path MTU issues**.
- **NodeLocal NAT** e **egress gateways** podem alterar origem do tráfego (SNAT); atenção a **ACLs** externas.
- **Reservas** e **exclusões** são úteis para interop com appliances/rotas existentes.
- Auditoria: validar **routes** por Node, **arp/nd**, e a convergência de **BGP/Cloud Routes** conforme o CNI.
- **Teste de esgotamento** controlado em ambiente de dev detecta falhas cedo.

**🔧 Comandos úteis:**
```bash
# Ver CIDRs de Service e Pod (variável conforme instalação)
kubectl cluster-info
kubectl describe node | egrep -i "podCIDR|podCIDRs"
kubectl get cm -n kube-system kube-proxy -o yaml | grep -i clusterCIDR -n

# Pod de teste + checagem de IP
kubectl run ipamtest --image=busybox:1.35 --restart=Never -- sleep 3600
kubectl get pod ipamtest -o wide
```

**📚 Referências:**
- Kubernetes – [IPv4/IPv6 Dual-Stack](https://kubernetes.io/docs/concepts/services-networking/dual-stack/)
- Kubernetes – [Services](https://kubernetes.io/docs/concepts/services-networking/service/)
- Kubernetes – [Network Plugins (IPAM nos CNIs)](https://kubernetes.io/docs/concepts/extend-kubernetes/compute-storage-net/network-plugins/)

---

## 10.4 DNS no Kubernetes
- O cluster fornece DNS interno para **Pods e Services** (padrão: `cluster.local`), resolvido pelo **CoreDNS**.
- **Services (ClusterIP)** ganham registros **A** do tipo `svc.ns.svc.cluster.local`; **headless** Services (`clusterIP: None`) publicam **A** por Pod.
- **Pods** ganham **A/AAAA** via headless Service ou registro reverso em alguns setups; para descoberta, prefira Services.
- **Search domains** no `/etc/resolv.conf` do Pod permitem resolver `my-svc` em `my-svc.my-ns.svc.cluster.local`.
- **ndots** influencia como nomes são consultados (muitos dots tentam FQDN primeiro, impactando latência).
- **StubDomains** e **upstreamNameservers** podem ser configurados no ConfigMap do CoreDNS (para resolver domínios externos).
- **NodeLocal DNSCache** melhora latência e reduz load do CoreDNS (daemonset local escutando em 169.254.20.10, por ex.).
- Debug: usar imagens como **netshoot** ou **busybox** e ferramentas `dig`, `nslookup`, `getent hosts`.
- O DNS integra-se aos **Services/EndpointSlices**: mudança de backends não altera o nome – aumenta resiliência.
- Atenção a **NetworkPolicies** que podem bloquear UDP/TCP 53 entre Pods e CoreDNS.
- **Split-horizon** e reescritas podem ser feitas com plugins do CoreDNS.
- Services `ExternalName` criam **CNAME** para nomes externos (sem VIP/Proxy).

**🔧 Exemplos:**
```bash
kubectl run dnsutils --image=registry.k8s.io/e2e-test-images/jessie-dnsutils:1.3 --restart=Never -- sleep 3600
kubectl exec -it dnsutils -- dig A kubernetes.default.svc.cluster.local
kubectl exec -it dnsutils -- nslookup my-svc.my-ns
```

**📚 Referências:**
- Kubernetes – [DNS for Services and Pods](https://kubernetes.io/docs/concepts/services-networking/dns-pod-service/)
- Kubernetes – [Customize DNS](https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/)
- Kubernetes – [NodeLocal DNSCache](https://kubernetes.io/docs/tasks/administer-cluster/nodelocaldns/)

---

## 10.5 CoreDNS no Kubernetes
- **CoreDNS** roda como **Deployment** em `kube-system` com um **ConfigMap** contendo o **Corefile** (cadeia de plugins).
- Plugin **`kubernetes`** responde por `*.svc.cluster.local` (ou domínio configurado), lendo Services/Endpoints via API.
- Plugins comuns: `forward` (encaminhar upstream), `cache`, `rewrite`, `ready`, `health`, `prometheus`, `log`.
- **Escalabilidade**: ajustar réplicas, `cache` e habilitar **NodeLocal DNSCache**; monitore latência e QPS no Prometheus.
- **StubDomains**: encaminham domínios específicos a DNS upstream distintos (ex.: corp.local → AD DNS).
- **rewrite** e **template** permitem flexibilizar respostas (cautela para não quebrar discovery de Services).
- **Observabilidade**: métricas em `/metrics` e logs detalhados com o plugin `log`.
- **Resolução externa**: `forward .` para resolvers do cluster/provedor; cuidado com **ndots/search** que podem gerar tentativas excessivas.
- **Segurança**: considere restringir quem pode editar o ConfigMap do CoreDNS; uma mudança errada derruba o DNS do cluster.
- **Affinity/Topology**: posicione réplicas em nós distintos para evitar SPOF local.
- **Teste/rollback**: altere o ConfigMap e monitore; tenha manifestos versionados para rollback rápido.
- **Compatibilidade**: versões do CoreDNS podem variar por distro/gerenciador (EKS/AKS/GKE).

**🔧 Manifests/Comandos:**
```bash
# Ver CoreDNS
kubectl -n kube-system get deploy,svc cm | grep -i coredns
kubectl -n kube-system get cm coredns -o yaml

# Exemplo: adicionando stub domain (trecho do Corefile)
# Corefile (parcial)
# .:53 {
#   kubernetes cluster.local in-addr.arpa ip6.arpa {
#     pods insecure
#     fallthrough in-addr.arpa ip6.arpa
#   }
#   forward corp.local 10.0.0.2 10.0.0.3
#   forward . /etc/resolv.conf
#   cache 30
# }
```

**📚 Referências:**
- Kubernetes – [Customize DNS Service](https://kubernetes.io/docs/tasks/administer-cluster/dns-custom-nameservers/)
- CoreDNS – [Plugins](https://coredns.io/plugins/)
- Kubernetes – [NodeLocal DNSCache](https://kubernetes.io/docs/tasks/administer-cluster/nodelocaldns/)

---

## 10.6 Ingress (L7 HTTP/HTTPS)
- **Ingress é um recurso** que define **regras L7** (host/path) para entrada de tráfego em Services do cluster.
- É necessário um **Ingress Controller** (NGINX, HAProxy, Traefik, ingress-gce, etc.); o recurso sozinho **não** faz nada.
- **IngressClass** seleciona o controlador; anotações e CRDs variam por implementação (timeouts, rewrites, rate-limit).
- **TLS** pode ser configurado (terminação no controller) referenciando **Secrets** com certificados.
- **Canary/blue-green** podem ser feitos com anotações/CRDs do controlador (ex.: nginx ingress canary).
- **Limitações**: originalmente focado em HTTP/HTTPS; para TCP/UDP use serviços L4 ou CRDs específicos do controller.
- **Service vs Ingress**: Service expõe Pods; Ingress **roteia** solicitações HTTP(S) para um ou mais Services.
- **NodePort/LoadBalancer** ainda são usados para **expor** o Ingress Controller ao exterior.
- Em clusters gerenciados, o controller pode **provisionar** Load Balancers automáticos.
- Observabilidade: métricas e logs do controller; atenção a **X-Forwarded-* headers**, sticky sessions, mTLS (se suportado).
- Segurança: WAF, restrições por IP, autenticação/OIDC via auth-url (varia por controller).
- Para rotas avançadas/multi-protocolos, considere **Gateway API**.

**🔧 Manifesto mínimo:**
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: web-ing
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  tls:
    - hosts: [ "app.example.com" ]
      secretName: app-tls
  rules:
    - host: app.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: web-svc
                port:
                  number: 80
```

**Comandos úteis:**
```bash
kubectl get ingress -A
kubectl describe ingress web-ing
kubectl logs -n ingress-nginx deploy/ingress-nginx-controller
```

**📚 Referências:**
- Kubernetes – [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)
- Kubernetes – [IngressClass](https://kubernetes.io/docs/concepts/services-networking/ingress/#ingress-class)

---

## 10.7 Gateway API (Sucessor evolutivo do Ingress)
- **Gateway API** é um conjunto de APIs (CRDs) pensado para **expressividade, portabilidade e separação de responsabilidades**.
- Recursos principais: **GatewayClass** (classe do provedor), **Gateway** (data plane/entradas), **HTTPRoute/TCPRoute/UDPRoute/GRPCRoute** (regras).
- **Multi-tenant**: permite **binding** de rotas a Gateways em **namespaces distintos**, com políticas de aprovação.
- **Listeners** em Gateway definem **portas, protocolos, TLS**, SNI e políticas; múltiplos **Routes** podem anexar-se a um listener.
- **Status/Conditions** ricos facilitam troubleshooting e automação (quem aceitou o quê, por quê).
- Suporta **TLS**, **mTLS**, políticas de **retry**, **time-outs**, **header mods**, **traffic splitting** (conforme controlador).
- **Ingress vs Gateway API**: Ingress é mais simples e amplamente suportado; Gateway API é **mais rico** e modular.
- Controladores de referência: **GKE/Gateway**, **Istio**, **Contour/Envoy**, **Kong**, **NGINX** (variando recursos suportados).
- **Gradual rollout**: é possível migrar rotas HTTP do Ingress para **HTTPRoute** mantendo equivalência.
- **Políticas adicionais** (RateLimit, Authn/Authz) surgem como **Policy APIs** que estendem rotas/gateways.
- **Observabilidade**: integra bem com métricas/telemetria (Envoy/Istio) e status condicionais.
- **Automação**: separa papéis – plataforma cria **Gateway**, times de app gerenciam **Routes**.
- **Compatibilidade**: verifique a **implementação** do seu provedor – nem tudo é GA em todos os ambientes.

**🔧 Manifesto de exemplo (HTTPRoute + Gateway):**
```yaml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: bank-gw
spec:
  gatewayClassName: nginx
  listeners:
  - name: https
    protocol: HTTPS
    port: 443
    tls:
      mode: Terminate
      certificateRefs:
      - name: bank-tls
---
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: app-route
spec:
  parentRefs:
  - name: bank-gw
  hostnames: ["app.example.com"]
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /
    backendRefs:
    - name: web-svc
      port: 80
```

**Comandos úteis:**
```bash
kubectl get gatewayclasses.gateway.networking.k8s.io
kubectl get gateways,gateway,httproutes -A
kubectl describe httproute app-route
```

**📚 Referências:**
- Kubernetes – [Gateway API (Visão Geral)](https://kubernetes.io/docs/concepts/services-networking/gateway/)
- Projeto – [Gateway API Docs](https://gateway-api.sigs.k8s.io/)
- Kubernetes – [Ingress vs Gateway](https://kubernetes.io/docs/concepts/services-networking/gateway/#comparison-with-ingress)

---

### 🧭 Dicas de prova & troubleshooting
> **Use sempre um Pod de utilidades** (ex.: `nicolaka/netshoot`): `curl`, `dig`, `ss`, `tcpdump` ajudam a diagnosticar DNS, rotas, latência e política.  
> **Valide objetos** com `kubectl describe` e **eventos** do namespace; inspecione logs do **kube-proxy**, **Ingress/Gateway controllers** e **CoreDNS**.

---

### 📑 Tabela express de comandos úteis
| Tema | Comandos |
|------|----------|
| Pod Net | `kubectl get pods -o wide -A` · `kubectl exec -it <pod> -- ip a` · `ip route` |
| DNS/CoreDNS | `kubectl -n kube-system get deploy coredns` · `kubectl exec -it dnsutils -- dig ...` |
| CNI | `ls /opt/cni/bin` · `ls /etc/cni/net.d` · `kubectl get daemonset -A | grep -i cni` |
| Services | `kubectl get svc -A` · `kubectl describe svc` · `kubectl get endpointslices -A` |
| Ingress | `kubectl get ingress -A` · `kubectl logs -n ingress-nginx deploy/ingress-nginx-controller` |
| Gateway API | `kubectl get gateway,httproute -A` · `kubectl describe gateway <name>` |

---


