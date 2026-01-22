## 🧠 What is a StatefulSet?
| Workload      | Controller  | Storage | Access |
| ------------- | ----------- | ------- | ------ |
| MySQL         | StatefulSet | EBS     | RWO    |
| MongoDB       | StatefulSet | EBS     | RWO    |
| Jenkins       | StatefulSet | EBS     | RWO    |
| Web uploads   | Deployment  | EFS     | RWX    |
| Static files  | Deployment  | EFS     | RWX    |
| Logs (shared) | DaemonSet   | EFS     | RWX    |


| Controller  | Storage | Access | Result   |
| ----------- | ------- | ------ | -------- |
| Pod         | EBS     | RWO    | OK       |
| Deployment  | EBS     | RWO    | ❌ BROKEN |
| StatefulSet | EBS     | RWO    | ✅ BEST   |
| Deployment  | EFS     | RWX    | ✅ BEST   |
| StatefulSet | EFS     | RWX    | ⚠️ Rare  |


| Workload | Controller  | PVC Count | Storage   |
| -------- | ----------- | --------- | --------- |
| NGINX    | Deployment  | 1         | EFS (RWX) |
| Web App  | Deployment  | 1         | EFS (RWX) |
| MySQL    | StatefulSet | replicas  | EBS (RWO) |
| Kafka    | StatefulSet | replicas  | EBS (RWO) |


| Action             | PVC | PV | EBS                |
| ------------------ | --- | -- | ------------------ |
| Delete Pod         | ✔️  | ✔️ | ✔️                 |
| Delete StatefulSet | ✔️  | ✔️ | ✔️                 |
| Delete PVC         | ❌   | ❌  | ❌ (Delete policy)  |
| Delete PVC         | ❌   | ✔️ | ✔️ (Retain policy) |



| Field                  | Why it matters          |
| ---------------------- | ----------------------- |
| `ebs.csi.aws.com`      | Correct CSI driver      |
| `WaitForFirstConsumer` | AZ-safe volume creation |
| `reclaimPolicy`        | Data retention behavior |
| `allowVolumeExpansion` | Online disk resize      |
| `gp3`                  | Best price/performance  |


| Situation                  | Old EBS reused?    |
| -------------------------- | ------------------ |
| StatefulSet deleted only   | ✅ YES              |
| StatefulSet + PVC deleted  | ❌ NO               |
| PVC deleted (Retain)       | ❌ NO (manual only) |
| Different StatefulSet name | ❌ NO               |
| Same name + PVC exists     | ✅ YES              |

## 🧠 What is a StatefulSet?

A **StatefulSet** is a Kubernetes workload used for **stateful applications** that require:

* Stable **pod names**
* Stable **network identity**
* Persistent **storage per pod**
* Ordered **start/stop/update**

Typical examples:

* Databases (MySQL, PostgreSQL)
* Kafka, Zookeeper
* Elasticsearch
* Redis (clustered mode)

---

## 🏗️ StatefulSet Architecture

![Image](https://miro.medium.com/v2/resize%3Afit%3A1200/1%2AHlgT4PgRsjrHj30vihI5Fw.png)

![Image](https://miro.medium.com/v2/resize%3Afit%3A1400/1%2AXNi3BmD6wfm3fvob7863Yw.png)

![Image](https://miro.medium.com/1%2AQxdrW189QuTeNF38T01lNg.png)

```
Headless Service
      │
      ▼
┌──────────── StatefulSet ────────────┐
│ pod-0 ─ PVC-0 ─ stable identity     │
│ pod-1 ─ PVC-1 ─ stable identity     │
│ pod-2 ─ PVC-2 ─ stable identity     │
└─────────────────────────────────────┘
```

---

## 🆚 StatefulSet vs Deployment (CRITICAL)

| Feature        | Deployment        | StatefulSet       |
| -------------- | ----------------- | ----------------- |
| Pod names      | Random            | Fixed (app-0,1,2) |
| Storage        | Shared / optional | Per-pod PVC       |
| Scaling order  | Parallel          | Ordered           |
| Rolling update | Fast              | Controlled        |
| Databases      | ❌                 | ✅                 |

---

## 📦 StatefulSet YAML (`statefulset.yml`)

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql
spec:
  serviceName: mysql-headless
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
        volumeMounts:
        - name: mysql-data
          mountPath: /var/lib/mysql
  volumeClaimTemplates:
  - metadata:
      name: mysql-data
    spec:
      accessModes: ["ReadWriteOnce"]
      resources:
        requests:
          storage: 10Gi
```

---

## 💾 Storage Behavior (VERY IMPORTANT)

Each pod gets **its own PVC**:

```
mysql-0 → pvc-mysql-data-mysql-0
mysql-1 → pvc-mysql-data-mysql-1
mysql-2 → pvc-mysql-data-mysql-2
```

✔ PVCs **are NOT deleted** when pod is deleted
✔ Data survives pod restarts
✔ Recreated pod re-attaches same PVC

---

## 🌐 Network Identity & DNS

Pods get **stable DNS names**:

```
mysql-0.mysql-headless.default.svc.cluster.local
mysql-1.mysql-headless.default.svc.cluster.local
```

Used for:

* Database replication
* Leader/follower setups
* Cluster discovery

---

## 🔄 Scaling Behavior

### Scale Up

```bash
kubectl scale statefulset mysql --replicas=4
```

Order:

```
mysql-3 → mysql-2 → mysql-1 → mysql-0
```

### Scale Down

```bash
kubectl scale statefulset mysql --replicas=1
```

Order:

```
mysql-2 → mysql-1 deleted
mysql-0 remains
```

⚠️ PVCs remain unless manually deleted

---

## 🗑️ Deletion Behavior (CRITICAL)

```bash
kubectl delete statefulset mysql
```

Result:

* Pods deleted ❌
* PVCs remain ✅
* Data preserved ✅

Delete PVC manually if needed:

```bash
kubectl delete pvc -l app=mysql
```

---

## 🔍 Verification Commands (SRE Style)

```bash
kubectl get statefulsets
kubectl get pods -o wide
kubectl get pvc
kubectl describe statefulset mysql
```

Check pod identity:

```bash
kubectl exec -it mysql-0 -- hostname
```

---

## 🚨 Common On-Call Issues

| Issue             | Cause                  |
| ----------------- | ---------------------- |
| Pod stuck Pending | PVC not bound          |
| Slow scaling      | Ordered rollout        |
| Data mismatch     | Wrong volumeMount      |
| Pod restart loops | App not stateful-aware |
| Storage full      | No volume expansion    |

Debug:

```bash
kubectl describe pod mysql-0
kubectl get events
```

---

## 🌍 Real-World Use Cases

| Application        | Why StatefulSet |
| ------------------ | --------------- |
| MySQL / PostgreSQL | Stable data     |
| Kafka              | Broker identity |
| Zookeeper          | Quorum          |
| Elasticsearch      | Shard ownership |
| Redis Cluster      | Slot mapping    |

---

## ✅ Production Best Practices

✔ Always use **Headless Service**
✔ One PVC per pod
✔ Never share PVCs
✔ Use **PodDisruptionBudget**
✔ Backups before scaling down
✔ Avoid StatefulSet for stateless apps
✔ Monitor disk usage

---

## 🧠 Interview-Ready One-Liners

* StatefulSet gives **stable identity**
* Pods are **ordered**
* PVCs survive pod deletion
* Headless Service is mandatory
* Databases ≠ Deployment

---

