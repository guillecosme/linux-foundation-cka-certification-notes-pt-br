
# 📘 KCNA - Resumo Completo de Preparação

Este documento contém uma revisão detalhada dos principais tópicos cobrados no exame **KCNA (Kubernetes and Cloud Native Associate)**, com explicações, exemplos, comandos úteis e links para a documentação oficial.

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



