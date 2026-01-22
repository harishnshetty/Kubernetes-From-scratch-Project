Below is a **real-world, production-oriented guide** to install **Kubernetes v1.34** using **kubeadm** on a **single Amazon EC2 Ubuntu server**.

> ⚠️ **Important reality check (production best practice)**
>
> * **Single EC2 = NOT true production HA** (no control-plane redundancy).
> * This setup is **production-grade configuration**, suitable for:
>
>   * POCs
>   * Edge clusters
>   * Internal tools
>   * Learning real-world ops
> * For real prod → **3 control planes + LB + separate worker nodes**

---

## 0️⃣ Prerequisites (AWS + OS)

### EC2

* Instance type: **t3.large or bigger** (2 vCPU, 8 GB RAM)
* OS: **Ubuntu 22.04 LTS**
* Disk: **50–100 GB gp3**
* Security Group:

  * SSH: `22`
  * Kubernetes API: `6443`
  * NodePort (optional): `30000–32767`

### System prep

```bash
sudo apt update && sudo apt upgrade -y
sudo apt-get install -y apt-transport-https ca-certificates curl gpg ncdu
sudo reboot
```

---

## 1️⃣ Disable Swap (MANDATORY)

```bash
sudo swapoff -a
sudo sed -i '/ swap / s/^/#/' /etc/fstab
```

sudo sed -i.bak 's|^/swap.img|#/swap.img|' /etc/fstab

sudo sed -i.bak '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab

sudo swapon --show && echo "---" && free -h

ssh harish@192.168.64.8 "sudo swapon --show; free -h"

sudo swapoff -a
sudo sed -i.bak '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
sudo systemctl restart kubelet

Verify:

```bash
free -h
```

---

## 2️⃣ Kernel Modules & Sysctl (Production-required)

```bash
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```
# Verify that the br_netfilter, overlay modules are loaded by running the following commands:
```bash
lsmod | grep br_netfilter
lsmod | grep overlay
```

Sysctl tuning: sysctl params required by setup, params persist across reboots

```bash
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
net.ipv4.ip_forward=1
EOF

sudo sysctl --system
```

---

## 3️⃣ Install Container Runtime (containerd – PROD STANDARD)

### Install containerd


https://github.com/containerd/containerd/releases 


```bash
curl -LO https://github.com/containerd/containerd/releases/download/v2.2.1/containerd-2.2.1-linux-amd64.tar.gz

sudo tar Cxzvf /usr/local containerd-2.2.1-linux-amd64.tar.gz

curl -LO https://raw.githubusercontent.com/containerd/containerd/main/containerd.service

sudo mkdir -p /usr/local/lib/systemd/system/
sudo mv containerd.service /usr/local/lib/systemd/system/
sudo mkdir -p /etc/containerd

containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

sudo systemctl daemon-reload
sudo systemctl enable --now containerd


# Check that containerd service is up and running
systemctl status containerd
```

Verify:

```bash
containerd --version
```

install runc
https://github.com/opencontainers/runc/releases

```bash
curl -LO https://github.com/opencontainers/runc/releases/download/v1.4.0/runc.amd64


sudo install -m 755 runc.amd64 /usr/local/sbin/runc
```


install cni plugin

https://github.com/containernetworking/plugins/


curl -LO https://github.com/containernetworking/plugins/releases/download/v1.9.0/cni-plugins-linux-amd64-v1.9.0.tgz
sudo mkdir -p /opt/cni/bin
sudo tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.9.0.tgz


---

## 4️⃣ Install kubeadm, kubelet, kubectl (v1.34)

### Add Kubernetes repo (new official method)
https://v1-34.docs.kubernetes.io/docs/setup/production-environment/tools/kubeadm/install-kubeadm/


```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.34/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.34/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

```

### Install components

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

Lock versions (PRODUCTION MUST)

1.34.3-1.1

Check:

```bash
kubeadm version
kubectl version
kubectl version --client
```

>Note: The reason we are installing 1.29, so that in one of the later task, we can upgrade the cluster to 1.35

8) Configure `crictl` to work with `containerd`
```bash
sudo crictl config runtime-endpoint unix:///var/run/containerd/containerd.sock
```
9) initialize control plane
```bash
sudo kubeadm init \
  --kubernetes-version=1.34.0 \
  --pod-network-cidr=192.168.0.0/16 \
  --apiserver-advertise-address=192.168.64.8 \
  --node-name master
```

sudo sed -i 's/disabled_plugins = \["cri"\]//g' /etc/containerd/config.toml

sudo systemctl restart containerd

# Cleanup the failed attempt
sudo kubeadm reset -f
sudo rm -rf ~/.kube
# Retry init
sudo kubeadm init --kubernetes-version=1.34.0 --pod-network-cidr=192.168.0.0/16 --apiserver-advertise-address=192.168.64.8 --node-name master

kubeadm join 192.168.64.8:6443 --token 9jpd4d.mtjhndj75rmrsjxz --discovery-token-ca-cert-hash sha256:87a1e393dff64952bcda1e7b05d7848fcc0736163fa438e84f8f7bba6f030761 


kubeadm token create --print-join-command

kubeadm join 192.168.64.8:6443 --token v1vcb1.x4r4peq3hvkmeu2s --discovery-token-ca-cert-hash sha256:87a1e393dff64952bcda1e7b05d7848fcc0736163fa438e84f8f7bba6f030761

If a node is already joined and you want to reuse it:
sudo kubeadm reset -f



## 6️⃣ Configure kubectl for Ubuntu User

```bash
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Test:

```bash
kubectl get nodes
```

---

## 7️⃣ Install CNI (Calico – Production-Grade)

https://docs.tigera.io/calico/latest/getting-started/kubernetes/self-managed-onprem/onpremises

```bash
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.3/manifests/operator-crds.yaml


kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.3/manifests/tigera-operator.yaml

curl -O https://raw.githubusercontent.com/projectcalico/calico/v3.31.3/manifests/custom-resources-bpf.yaml


kubectl apply -f custom-resources-bpf.yaml

watch kubectl get tigerastatus
```

Wait:

```bash
kubectl get pods -n kube-system
```

Node should become:

```bash
kubectl get nodes
```

---

## 8️⃣ Allow Scheduling Pods on Control Plane (Single Node Only)

```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

---

## 9️⃣ Production Hardening (IMPORTANT)

### Enable kubelet

```bash
sudo systemctl enable kubelet
```

### Firewall (optional but recommended)

```bash
sudo ufw allow 22
sudo ufw allow 6443
sudo ufw enable
```

---

## 🔟 Validation (Production sanity checks)

```bash
kubectl get nodes -o wide
kubectl get pods -A
crictl ps
```

Test workload:

```bash
kubectl create deployment nginx --image=nginx
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl get svc
```

---

## 🏗️ Real-World Production Notes (VERY IMPORTANT)

### ❌ What this setup is NOT

* No HA
* No etcd backup strategy
* No autoscaling
* No LB

### ✅ What real production adds

| Area             | Tool                  |
| ---------------- | --------------------- |
| HA control plane | 3 EC2 nodes           |
| Load balancer    | NLB / HAProxy         |
| Storage          | EBS CSI / EFS CSI     |
| Ingress          | NGINX / Gateway API   |
| Security         | NetworkPolicies, RBAC |
| Monitoring       | Prometheus + Grafana  |
| Backup           | Velero                |
| GitOps           | Argo CD               |

---

Perform the below steps on both the worker nodes
Perform steps 1-8 on both the nodes
Run the command generated in step 9 on the Master node which is similar to below
sudo kubeadm join 172.31.71.210:6443 --token xxxxx --discovery-token-ca-cert-hash sha256:xxx
If you forgot to copy the command, you can execute below command on master node to generate the join command again
kubeadm token create --print-join-command