## 🧠 Kubernetes Workloads – Overview

Kubernetes workloads define **how containers run** inside the cluster.

| Workload          | Purpose                   |
| ----------------- | ------------------------- |
| Pod               | Smallest deployable unit  |
| ReplicaController | Legacy replication        |
| ReplicaSet        | Replica management        |
| Deployment        | Rolling updates & scaling |
| Job               | One-time task             |
| CronJob           | Scheduled task            |
| DaemonSet         | Runs on every node        |
| Init Container    | Pre-run setup             |
| Sidecar           | Supporting container      |

---

## 🏗️ Kubernetes Workloads Architecture

![Image](https://www.cherryservers.com/v3/assets/blog/2025-05-19/img-01.png)

![Image](https://storage.googleapis.com/cdn.thenewstack.io/media/2017/11/07751442-deployment.png)

![Image](https://miro.medium.com/1%2A-1Bz713XTxzs3KBi1lEAXg.png)

```
┌─────────────┐
│   kubectl   │
└─────┬───────┘
      │
┌─────▼──────────────────────────────┐
│ Kubernetes Control Plane           │
│ ├─ Deployment → ReplicaSet → Pod   │
│ ├─ Job → Pod                        │
│ ├─ CronJob → Job → Pod             │
│ ├─ DaemonSet → Pod (per node)      │
└────────────────────────────────────┘
```

---

## 📦 Workload Types Explained (With YAML)

---

### 🔹 1. Pod (`pod.yml`)

**Smallest unit in Kubernetes**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: demo-pod
spec:
  containers:
  - name: nginx
    image: nginx
```

✔ Not self-healing
❌ Not recommended for production

---

### 🔹 2. ReplicaController (`replicacontoller.yml`)

> ⚠️ **Deprecated** – replaced by ReplicaSet

```yaml
kind: ReplicationController
spec:
  replicas: 2
```

✔ Legacy clusters only

---

### 🔹 3. ReplicaSet (`replicaset.yml`)

Ensures **fixed number of pods**

```yaml
kind: ReplicaSet
spec:
  replicas: 3
```

✔ Used internally by Deployments

---

### 🔹 4. Deployment (`deployment.yml`)

**Most common production workload**

```yaml
kind: Deployment
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
```

✔ Rolling updates
✔ Rollbacks
✔ Auto-healing

---

### 🔹 5. Job (`job.yml`)

Runs **once and exits**

```yaml
kind: Job
spec:
  completions: 1
```

✔ DB migrations
✔ Backup jobs

---

### 🔹 6. CronJob (`cron-job.yml`)

Scheduled Jobs (Linux cron style)

```yaml
kind: CronJob
spec:
  schedule: "*/5 * * * *"
```

✔ Log cleanup
✔ Reports
✔ Batch processing

---

### 🔹 7. DaemonSet (`daemonset.yml`)

Runs **one pod per node**

```yaml
kind: DaemonSet
```

✔ Log collectors
✔ Monitoring agents
✔ Security agents

---

### 🔹 8. Init Container (`init-container.yml`)

Runs **before app container starts**

```yaml
initContainers:
- name: init-db
  image: busybox
```

✔ DB wait
✔ Config generation

---

### 🔹 9. Sidecar Container (`sidecar.yml`)

Runs **alongside main container**

```yaml
containers:
- name: app
- name: sidecar
```

✔ Logging
✔ Proxy
✔ Metrics exporter

---

## 🔄 Execution Flow (Very Important)

```
Init Container → App Container + Sidecar
                    │
              Liveness / Readiness
```

```
CronJob → Job → Pod → Complete → Exit
```

---

## 🚀 Apply All Manifests

```bash
kubectl apply -f pod.yml
kubectl apply -f replicaset.yml
kubectl apply -f deployment.yml
kubectl apply -f job.yml
kubectl apply -f cron-job.yml
kubectl apply -f daemonset.yml
kubectl apply -f init-container.yml
kubectl apply -f sidecar.yml
```

---

## 🔍 Verification (SRE Commands)

```bash
kubectl get pods
kubectl get deploy
kubectl get rs
kubectl get jobs
kubectl get cronjobs
kubectl get ds
```

Describe deeply:

```bash
kubectl describe pod <pod-name>
```

---

## 🌍 Real-World Use Cases

| Scenario           | Workload       |
| ------------------ | -------------- |
| Web App            | Deployment     |
| Database Migration | Job            |
| Daily Backup       | CronJob        |
| Logging Agent      | DaemonSet      |
| App Bootstrap      | Init Container |
| Log Shipping       | Sidecar        |

---

## ✅ Best Practices (Production)

✔ Never use Pods directly in prod
✔ Always use Deployments
✔ Prefer CronJob over OS cron
✔ DaemonSet for node-level tasks
✔ Init containers for dependencies
✔ Sidecars for observability

