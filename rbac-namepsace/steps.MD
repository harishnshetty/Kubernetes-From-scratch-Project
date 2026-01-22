You’re mixing **RBAC**, **key generation**, **CSR**, **certificate approval**, and **renewal** steps together — totally normal confusion 👍
Let’s **reset everything** and I’ll explain it **cleanly, step-by-step**, including **what to regenerate and when**.

---

# 🔐 Kubernetes User Authentication (Certificate-based) – Proper Flow

Kubernetes **users are external** (not Kubernetes objects).
Authentication is done using **X.509 certificates**, and **authorization** is done using **RBAC**.

---

## 🧠 Big Picture (Important)

| Component                   | Purpose                                            |
| --------------------------- | -------------------------------------------------- |
| **Private Key (.key)**      | Proves identity (never changes unless compromised) |
| **CSR (.csr)**              | Request to Kubernetes CA                           |
| **Certificate (.crt)**      | Signed identity (expires, must be renewed)         |
| **RBAC (Role/RoleBinding)** | What the user is allowed to do                     |

➡️ **Key ≠ Certificate**
➡️ **Key is long-lived**
➡️ **Certificate expires and gets renewed**

---

# ✅ STEP 1: Generate Private Key (ONE TIME)

🔑 This is **your identity root**.
❌ Do **NOT regenerate unless compromised**

```bash
openssl genrsa -out harish.key 2048
```

✔ Output:

```
harish.key
```

---

# ✅ STEP 2: Generate CSR (Whenever you need a new cert)

CSR = “Hey Kubernetes, please sign this key”

```bash
openssl req -new \
  -key harish.key \
  -out harish.csr \
  -subj "/CN=harish/O=dev-team"
```

### What matters:

| Field | Meaning             |
| ----- | ------------------- |
| CN    | Username (`harish`) |
| O     | Group (`dev-team`)  |

✔ Kubernetes uses:

* **CN → User**
* **O → Group**

---

# ✅ STEP 3: Base64 Encode CSR

```bash
cat harish.csr | base64 | tr -d '\n'
```

Copy the output.

---

# ✅ STEP 4: Create Kubernetes CSR Object

📄 `csr.yaml`

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: harish-csr
spec:
  request: <PASTE_BASE64_CSR_HERE>
  signerName: kubernetes.io/kube-apiserver-client
  usages:
    - client auth
```

Apply it:

```bash
kubectl apply -f csr.yaml
```

---

# ✅ STEP 5: Approve CSR (Admin Only)

```bash
kubectl get csr
kubectl certificate approve harish-csr
```

⛔ **Do NOT approve & deny both**

* ✔ approve → cert issued
* ❌ deny → request rejected (delete & recreate)

---

# ✅ STEP 6: Extract the Signed Certificate

```bash
kubectl get csr harish-csr -o jsonpath='{.status.certificate}' \
| base64 --decode > harish.crt
```

✔ Now you have:

```
harish.key   (private)
harish.crt   (certificate)
```

---

# ✅ STEP 7: Verify Certificate Expiry

```bash
openssl x509 --noout -dates -in harish.crt
```

Example:

```
notBefore=Jan 15 05:00:00 2026 GMT
notAfter=Jan 15 05:00:00 2027 GMT
```

⏳ Default validity: **1 year**

---

# ✅ STEP 8: Create Role (If Not Exists)

📄 `role.yaml`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

```bash
kubectl apply -f role.yaml
```

---

# ✅ STEP 9: Bind User to Role (Your RoleBinding)

📄 `rolebinding.yaml`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: harish
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl apply -f rolebinding.yaml
```

---

# ✅ STEP 10: Configure kubeconfig for User

```bash
kubectl config set-credentials harish \
  --client-certificate=harish.crt \
  --client-key=harish.key
```

## (CREATE / UPDATE)
```bash
kubectl config set-context harish-context \
  --cluster=minikube \
  --user=harish
```
## (SWITCH)
```bash
kubectl config use-context harish-context
```

```bash
kubectl config use-context minikube
```

```bash
kubectl config get-contexts
```

Test:

```bash
kubectl get pods
```

---

# 🔁 CERTIFICATE RENEWAL (THIS IS KEY 🔑)

## ❓ What expires?

✔ Certificate (`.crt`)
❌ Private Key (`.key`) does NOT

---

## 🔄 Renewal Steps (Simple)

When cert expires:

```bash
# Reuse SAME key
openssl req -new -key harish.key -out harish.csr -subj "/CN=harish/O=dev-team"
```

Then repeat:

* CSR yaml
* kubectl apply
* approve
* extract new `harish.crt`

🎯 **No RBAC changes required**

---

# ❌ When do you regenerate the KEY?

Only if:

* Key leaked
* Laptop compromised
* Security rotation required

Otherwise:
❌ Never regenerate key

---

# 🧠 FINAL CHEAT SHEET

| Action         | Needed                |
| -------------- | --------------------- |
| Generate key   | ✅ once                |
| Generate CSR   | 🔁 every cert         |
| Approve CSR    | 🔁 every cert         |
| Renew cert     | 🔁 yearly             |
| Recreate RBAC  | ❌ no                  |
| Regenerate key | ❌ only if compromised |

---

If you want, I can also show:

* **Group-based RBAC**
* **Multiple namespaces**
* **Short-lived certs**
* **EKS / kubeadm difference**
* **Automated cert rotation**

Just tell me 👍

```bash
kubectl config use-context minikube
```

```bash
kubectl config use-context kubernetes-admin@kubernetes
```
