kubectl apply -f role.yml
kubectl apply -f binding.yml
kubectl get pods --as harish

kubectl auth can-i list pods --as harish

kubectl auth can-i create pods --as harish

kubectl auth can-i create pods --as harish --namespace kube-system

kubectl auth can-i create pods --as harish --namespace default

kubectl auth can-i create pods --as harish --namespace default --as-group system:bootstrappers

kubectl auth can-i create pods --as harish --namespace default --as-group system:bootstrappers --as-group system:authenticated

kubectl auth can-i create pods --as harish --namespace default --as-group system:bootstrappers --as-group system:authenticated --as-group system:nodes  

kubectl auth can-i create pods --as harish --namespace default --as-group system:bootstrappers --as-group system:authenticated --as-group system:nodes --as-group system:nodes-bootstrapper

kubectl auth can-i create pods --as harish --namespace default --as-group system:bootstrappers --as-group system:authenticated --as-group system:nodes --as-group system:nodes-bootstrapper --as-group system:serviceaccounts

kubectl auth can-i create pods --as harish --namespace default --as-group system:bootstrappers --as-group system:authenticated --as-group system:nodes --as-group system:nodes-bootstrapper --as-group system:serviceaccounts --as-group system:serviceaccounts:default

kubectl auth can-i create pods --as harish --namespace default --as-group system:bootstrappers --as-group system:authenticated --as-group system:nodes --as-group system:nodes-bootstrapper --as-group system:serviceaccounts --as-group system:serviceaccounts:default --as-group system:serviceaccounts:system


kubectl auth can-i get pod

kubectl auth whoami

kubectl get role

kubectl get rolebinding

kubectl get role -A --no-headers | wc -l

kubectl describe rolebinding read-pods -n read-pods



kubectl config set-credentials harish --client-certificate=harish.crt --client-key=harish.key --embed-certs=true 

kubectl config set-context harish-context --cluster=minikube --user=harish --namespace=default

kubectl config get-contexts

kubectl config use-context harish-context

kubectl get pods --as harish

kubectl auth can-i list pods --as harish




