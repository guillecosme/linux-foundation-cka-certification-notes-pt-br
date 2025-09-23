
# 📘 KCNA - Resumo Completo de Preparação

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
Todos os componentes críticos do Kubernetes (API Server, etcd, Controller Manager e Kubelets) se comunicam via **TLS**, garantindo criptografia e autenticação mútua.  
Isso protege contra ataques de interceptação de tráfego e garante que apenas componentes legítimos possam se conectar. O administrador precisa garantir a renovação periódica dos certificados, geralmente localizados em `/etc/kubernetes/pki/`.  
Erros comuns incluem certificados expirados ou emitidos para nomes incorretos. É essencial auditar regularmente esses certificados.

📖 Doc: [Certificates](https://kubernetes.io/docs/tasks/tls/)

**Exemplo:** verificar certificados usados pelo API Server:
```bash
openssl s_client -connect 127.0.0.1:6443 -showcerts
```

---

## 8.4. 🧾 Certificados
Kubernetes utiliza certificados x509 extensivamente para autenticar componentes internos e usuários. Cada parte do cluster (apiserver, controller, scheduler, kubelet, etcd) possui certificados próprios.  
Eles podem ser gerenciados manualmente ou via **kubeadm**, que cria e renova automaticamente. Um problema comum é o vencimento desses certificados, que pode derrubar o cluster.  
Para ambientes críticos, é recomendado usar uma CA corporativa ou um sistema automatizado de rotação de certificados.

📖 Doc: [PKI Certificates and Requirements](https://kubernetes.io/docs/setup/best-practices/certificates/)

**Exemplo:** gerar CSR (Certificate Signing Request) com OpenSSL:
```bash
openssl req -new -newkey rsa:2048 -nodes -keyout apiserver.key -out apiserver.csr
```

---

## 8.5. ⚙️ KubeConfig
O arquivo `kubeconfig` define como usuários e ferramentas se conectam ao cluster. Ele contém informações como **clusters, usuários, contextos e credenciais**.  
Manter esse arquivo seguro é fundamental, pois quem tiver acesso pode operar o cluster com as permissões configuradas. Recomenda-se configurar múltiplos contextos para separar ambientes (dev, stage, prod).  
Além disso, boas práticas incluem limitar o tempo de validade dos tokens e evitar que chaves privadas fiquem expostas em sistemas de versionamento.

📖 Doc: [Organizing Cluster Access Using kubeconfig Files](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/)

**Exemplo:** trocar contexto ativo para ambiente de produção:
```bash
kubectl config use-context prod-admin
```

---

## 8.6. 📚 API Groups
A API do Kubernetes é dividida em **grupos** que organizam recursos de forma lógica. O grupo **Core (v1)** contém recursos básicos como Pods e Services, enquanto grupos como `apps/v1` e `rbac.authorization.k8s.io/v1` abrangem funcionalidades mais avançadas.  
Entender os grupos de API é essencial para criar regras de RBAC e YAMLs corretos. Além disso, conhecer as versões disponíveis ajuda a evitar recursos obsoletos.  
Para o exame CKA, é importante saber diferenciar recursos Core de recursos que pertencem a grupos específicos.

📖 Doc: [Kubernetes API Overview](https://kubernetes.io/docs/reference/using-api/)

**Exemplo:** listar todas as versões de API disponíveis:
```bash
kubectl api-versions
```

---

## 8.7. 👤 RBAC Roles
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

## 8.8. 🌐 ClusterRoles
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

## 8.9. 🧑‍💻 Service Accounts
ServiceAccounts fornecem identidade para Pods se autenticarem na API.  
Cada Pod por padrão recebe uma conta de serviço no namespace default, mas é uma boa prática criar ServiceAccounts dedicadas para cada aplicação.  
Isso permite aplicar RBAC específico para workloads, evitando que aplicações tenham mais acesso que o necessário. Tokens de ServiceAccounts podem ser montados como volumes ou injetados automaticamente em Pods.

📖 Doc: [Configure Service Accounts for Pods](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/)

**Exemplo:** criar uma ServiceAccount:
```bash
kubectl create serviceaccount my-sa -n secure-apps
```

---

## 8.10. 🖼️ Image Security
Manter segurança em imagens de containers é essencial. Use sempre imagens de registries confiáveis, mantenha versões atualizadas e verifique assinaturas quando disponíveis.  
Além disso, configure `imagePullSecrets` para acessar registries privados com segurança. Ferramentas como Trivy podem ser usadas para escanear imagens em busca de vulnerabilidades.  
Evite usar a tag `latest`, pois isso dificulta auditorias e pode causar inconsistência entre ambientes.

📖 Doc: [Image Pull Secrets](https://kubernetes.io/docs/concepts/containers/images/)

**Exemplo:** criar secret para autenticação em registry privado:
```bash
kubectl create secret docker-registry regcred   --docker-server=myregistry.io   --docker-username=user   --docker-password=pass   --docker-email=user@example.com
```

---

## 8.11. 🐳 Security nos Docker Containers
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

## 8.12. 🌐 Network Policies
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

## 8.13. 🔄 Kubectx e Kubens
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

