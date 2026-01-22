# 🚀 Kubernetes Health Checks (Liveness & Readiness)

> Production-grade examples of **Liveness** and **Readiness** probes using
> **exec**, **HTTP**, and **TCP** methods with real YAML manifests.

---

## 📑 Table of Contents

* What are Probes?
* Liveness vs Readiness
* Architecture Diagram
* Probe Types
* YAML Examples
* Deployment Order
* Verification Commands
* Failure Scenarios
* Best Practices

---

## 🧠 What are Kubernetes Probes?

Kubernetes probes allow the **kubelet** to check container health.

| Probe Type    | Purpose                                  |
| ------------- | ---------------------------------------- |
| **Liveness**  | Is the container alive? If not → restart |
| **Readiness** | Is the container ready to serve traffic? |
| **Startup**   | Is the app finished starting?            |

---

## 🔍 Liveness vs Readiness (Real Meaning)

```
Liveness  → restart container
Readiness → remove pod from Service endpoints
```

| Scenario     | Liveness  | Readiness    |
| ------------ | --------- | ------------ |
| App crashed  | ❌ restart | ❌ no traffic |
| DB down      | ✅ running | ❌ no traffic |
| Slow startup | ❌ restart | ❌ no traffic |

---

## 🏗️ Architecture Diagram – Probe Flow

![Image](https://developers.redhat.com/sites/default/files/blog/2020/07/Probes-Example4-Solution.png)

![Image](https://cdn.prod.website-files.com/635e4ccf77408db6bd802ae6/6716204cc887a70bfc5c063e_AD_4nXdzBlMJxmsQsOnfVyAt-q-kyyPfQ2jyed2JrLNeDdIwMVIj9PKqP8eqhLGpOa_Wvh7bQgA4GBSdDxaq1PoDHtF0hBBeeieuOfIQTpuMN4iVQcMChAeB8lhifZ26_5B6uXhWDxDQSRodnYqOLdoP6gcFkI6isWiedFes2YESFg.gif)

```
┌──────────────┐
│   Service    │
└──────┬───────┘
       │ (only Ready pods)
┌──────▼─────────────────────┐
│ Pod                         │
│ ┌──────── Container ──────┐ │
│ │ Liveness Probe           │ │
│ │ Readiness Probe          │ │
│ └─────────────────────────┘ │
└───────────┬─────────────────┘
            │
        Kubelet
```

---

## 🔧 Probe Types Supported

| Probe     | Method | Example Use         |
| --------- | ------ | ------------------- |
| Liveness  | exec   | Check process/file  |
| Liveness  | HTTP   | App health endpoint |
| Liveness  | TCP    | DB / socket check   |
| Readiness | HTTP   | API ready check     |

---

## 📦 YAML Examples

---

### 🔹 1. Liveness Probe – Command (`liveness-command.yml`)

```yaml
livenessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 10
  periodSeconds: 5
```

✔ Used when:

* Checking file existence
* Checking running process
* Lightweight checks

---

### 🔹 2. Liveness Probe – HTTP (NGINX) (`liveness-http-nginx.yml`)

```yaml
livenessProbe:
  httpGet:
    path: /
    port: 80
  initialDelaySeconds: 5
  periodSeconds: 10
```

✔ Best for:

* Web servers
* REST APIs

---

### 🔹 3. Liveness Probe – HTTP Failure (`liveness-http-error.yml`)

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  failureThreshold: 3
```

❌ If endpoint returns **500 / timeout** → container restarts

---

### 🔹 4. Liveness Probe – TCP (`liveness-tcp.yml`)

```yaml
livenessProbe:
  tcpSocket:
    port: 3306
  initialDelaySeconds: 15
  periodSeconds: 20
```

✔ Ideal for:

* MySQL
* Redis
* Kafka
* Any TCP service

---

### 🔹 5. Readiness Probe – HTTP (`readness-http.yml`)

```yaml
readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

✔ Pod receives traffic **ONLY after this passes**

---

## 🚀 Apply Manifests

```bash
kubectl apply -f liveness-command.yml
kubectl apply -f liveness-http-nginx.yml
kubectl apply -f liveness-http-error.yml
kubectl apply -f liveness-tcp.yml
kubectl apply -f readness-http.yml
```

---

## 🔎 Verification Commands (SRE Style)

```bash
kubectl get pods
kubectl describe pod <pod-name>
kubectl get events --sort-by=.metadata.creationTimestamp
```

Watch probe failures live:

```bash
kubectl logs <pod-name> --previous
```

---

## 💥 Failure Scenarios (Important!)

| Failure             | Result                   |
| ------------------- | ------------------------ |
| Liveness fails      | Container restarts       |
| Readiness fails     | Pod removed from Service |
| Both fail           | Restart + no traffic     |
| Probe misconfigured | CrashLoopBackOff         |

---

## ✅ Best Practices (Production)

✔ Always use **Readiness** with Services
✔ Keep Liveness **lightweight**
✔ Avoid DB checks in Liveness
✔ Use StartupProbe for slow apps
✔ Tune `initialDelaySeconds` carefully
✔ Monitor with Prometheus alerts

---

## 🧭 Real-World Recommendation

| App Type    | Recommended               |
| ----------- | ------------------------- |
| API         | HTTP Liveness + Readiness |
| Database    | TCP Liveness              |
| Legacy App  | exec probe                |
| Spring Boot | `/actuator/health`        |

---
