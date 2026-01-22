
### **CoreDNS**

**What it does:**
Provides **DNS service inside the Kubernetes cluster** so Pods can find each other by name.
CoreDNS is the **internal DNS of Kubernetes**.
It allows **Pods and Services to find and talk to each other using names instead of IPs**, which constantly change.

**Example:**
`frontend` Pod connects to `backend.default.svc.cluster.local` without knowing IPs.


---

## ✅ Real-World Use Case: Microservices Communication

### Scenario

You have an **E-commerce application** running on Kubernetes:

* `frontend` (React)
* `backend` (Node.js / Java API)
* `database` (MongoDB)

Each component runs in different Pods, and Pods can restart at any time → **IP addresses change**.

---

### ❌ Without CoreDNS (problem)

* Backend Pod IP changes after restart
* Frontend still points to old IP
* Application breaks ❌

---

### ✅ With CoreDNS (solution)

You create a Kubernetes **Service** for backend:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend
spec:
  selector:
    app: backend
  ports:
  - port: 3000
```

CoreDNS automatically creates a DNS entry:

```
backend.default.svc.cluster.local
```

---

### 🔄 How traffic flows

1. Frontend sends request to:

   ```
   http://backend:3000
   ```
2. CoreDNS resolves `backend` → Service IP
3. Service load-balances traffic to backend Pods
4. Application works even if Pods restart ✅

---

## 📌 Another Real-World Example: Database Access

Backend connects to database using DNS:

```env
DB_HOST=mongodb.default.svc.cluster.local
```

Even if MongoDB Pods restart or scale, **no config change needed**.

---

## 🚀 Enterprise Use Case

* Used in **microservices, EKS, GKE, AKS**
* Mandatory for **service discovery**
* Critical for **zero-downtime deployments**
* Required for **HPA, Ingress, Gateway API**

## 🎯 Interview One-Liner

**CoreDNS enables service discovery in Kubernetes by resolving service names to IPs, allowing microservices to communicate reliably even when Pods change.**



---

### **storage-provisioner**

**What it does:**
Automatically **creates Persistent Volumes (PV)** when a PVC is requested (Dynamic Provisioning).

**Example:**
PVC → automatically creates an EBS volume in AWS or local disk in Minikube.

---

### **ngf-nginx-gateway-fabric**

**What it does:**
An **NGINX-based Gateway API implementation** for managing traffic using Kubernetes Gateway API.

**Example:**
Routes HTTP traffic to multiple services using `Gateway` and `HTTPRoute` instead of Ingress.

---

### **kubelet**

**What it does:**
Runs on every worker node and **ensures Pods and containers are running** as defined.

**Example:**
Starts containers, reports Pod status to API Server, restarts failed containers.

---

### **kube-proxy**

**What it does:**
Handles **network routing and load balancing** for Kubernetes Services.

**Example:**
A request to `ClusterIP` is forwarded to one of the backend Pods.

---

### **kube-scheduler**

**What it does:**
Decides **which node a Pod should run on** based on resources and constraints.

**Example:**
Schedules a Pod on the node with enough CPU and memory.

---

### **kube-controller-manager**

**What it does:**
Runs controllers that **maintain the desired state** of the cluster.

**Example:**
If a Pod crashes, the ReplicaSet controller creates a new Pod automatically.

---

### **kube-apiserver**

**What it does:**
The **main entry point** of the Kubernetes cluster; all requests go through it.

**Example:**
`kubectl get pods` → API Server → etcd → response returned.

---

### **etcd**

**What it does:**
A **distributed key-value store** that stores all cluster state and configuration.

**Example:**
Stores Pod specs, secrets, ConfigMaps, and node information.

---

### **kindnet**

**What it does:**
A **CNI (Container Network Interface) plugin** used by KIND clusters for Pod networking.

**Example:**
Assigns IPs to Pods and enables Pod-to-Pod communication.

---

### **metrics-server**

**What it does:**
Collects **CPU and memory usage metrics** from nodes and Pods.

**Example:**
Used by `kubectl top pods` and Horizontal Pod Autoscaler (HPA).

---

### 🔁 **storage-provisioner** (duplicate)

Same as above – enables **dynamic storage creation** for PVCs.

---

### 🔁 **ngf-nginx-gateway-fabric** (duplicate)

Same as above – provides **Gateway API-based traffic management** using NGINX.



---

## 🧠 **Control Plane Components**

(Manage and control the Kubernetes cluster)

* **kube-apiserver** – Central API entry point for all cluster operations.
* **etcd** – Distributed key-value store holding cluster state and configuration.
* **kube-scheduler** – Assigns Pods to suitable worker nodes.
* **kube-controller-manager** – Runs controllers to maintain desired cluster state.

---

## 🖥️ **Node (Worker) Components**

(Run applications and handle networking on worker nodes)

* **kubelet** – Ensures Pods and containers are running as defined.
* **kube-proxy** – Handles Service networking and load balancing.
* **kindnet (CNI)** – Provides Pod-to-Pod networking and IP assignment.

---

## 🧩 **Add-ons / Cluster Services**

(Extend Kubernetes functionality)

* **CoreDNS** – Provides DNS-based service discovery inside the cluster.
* **metrics-server** – Collects CPU and memory metrics for Pods and nodes.
* **storage-provisioner** – Enables dynamic Persistent Volume provisioning.
* **ngf-nginx-gateway-fabric** – Implements Gateway API using NGINX for traffic routing.

---

