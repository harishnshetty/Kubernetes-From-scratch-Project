# Kubernetes-From-scratch-Project

nano ~/.zshrc



alias c='clear'
alias k='kubectl'
alias kn='kubectl config set-context --current --namespace'

source ~/.zshrc
```bash
| Gateway Implementation   | controllerName                                   |
| ------------------------ | ------------------------------------------------ |
| **AWS ALB Gateway**      | `eks.amazonaws.com/aws-load-balancer-controller` |
| **NGINX Gateway Fabric** | `gateway.nginx.org/nginx-gateway-controller`     |
| Istio                    | `istio.io/gateway-controller`                    |
| Kong                     | `konghq.com/kic-gateway-controller`              |





```bash
kubectl apply --server-side -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.4.1/standard-install.yaml
```