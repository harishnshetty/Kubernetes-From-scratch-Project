kubectl drain master --ignore-daemonsets

kubectl drain worker1 --ignore-daemonsets

kubectl drain worker2 --ignore-daemonsets

kubectl cordon master


kubeadm version


ssh control-plane

kubeadm upgrade -h
kubeadm upgrade plan

kubeadm upgrade apply v1.29.6

apt-mark hold kubelet kubeadm kubectl

apt-get update && apt-get install -y kubelet=1.29.6-1.1 kubeadm=1.29.6-1.1 kubectl=1.29.6-1.1 --allow-downgrades --allow-change-held-packages

apt-mark hold kubelet kubeadm kubectl

