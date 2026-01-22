
### **CoreDNS**

**What it does:**
Provides **DNS service inside the Kubernetes cluster** so Pods can find each other by name.

**Example:**
`frontend` Pod connects to `backend.default.svc.cluster.local` without knowing IPs.

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
