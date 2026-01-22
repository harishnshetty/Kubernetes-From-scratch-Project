| Feature             | Minikube | Real-world |
| ------------------- | -------- | ---------- |
| StorageClass        | ✅ Yes    | ✅ Yes      |
| Dynamic PVC         | ✅ Yes    | ✅ Yes      |
| StatefulSet + PVC   | ✅ Yes    | ✅ Yes      |
| Volume expansion    | ✅ Yes    | ✅ Yes      |
| ReclaimPolicy       | ✅ Yes    | ✅ Yes      |
| AccessModes         | ✅ Yes    | ✅ Yes      |
| CSI driver concepts | ⚠️ Local | EBS / EFS  |


kubectl get storageclass
