calico plugin setup


minikube addons enable ingress


minikube start \
  --network-plugin=cni \
  --cni=calico

kubectl get pods -n kube-system


kubectl api-resources | grep networkpolicy


https://docs.tigera.io/calico/latest/getting-started/kubernetes/minikube

minikube start --network-plugin=cni

kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.31.3/manifests/calico.yaml

watch kubectl get pods -l k8s-app=calico-node -A



kubectl logs my-sql-0 --previous


kubectl exec -it my-sql-0 -- mysql -u root -p


kubectl get ingress myingress

curl -v --resolve app.myhost.com:80:192.168.49.2 http://app.myhost.com/

kubectl get svc -A

curl -v --resolve app.myhost.com:80:127.0.0.1 http://app.myhost.com/

curl --resolve app.myhost.com:8081:127.0.0.1 http://app.myhost.com:8081/







kubectl exec frontend-78cf5c94bc-t4br4 -- curl -v http://backend-svc:4000/health
kubectl exec frontend-78cf5c94bc-t4br4 -- curl -v http://backend-svc:4000/transaction
kubectl exec frontend-78cf5c94bc-qkv6d -- nc -zv my-sql 3306 --> wrong


kubectl exec backend-65b6c59887-7f69g -- nc -zv my-sql 3306  --> right


kubectl get networkpolicy


apk update
apk add telnet


kubectl exec -it frontend-78cf5c94bc-qkv6d -- sh
curl -v http://backend-svc:4000/health

