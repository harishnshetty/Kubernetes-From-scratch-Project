CSR is used to securely authenticate nodes and users in Kubernetes by issuing trusted TLS certificates signed by the cluster CA.

Yes, you can access a Kubernetes cluster using only a CSR if certificate authentication is enabled and RBAC permissions are granted — but in EKS this requires an existing admin and cannot replace IAM-based access.



openssl genrsa -out harish.key 2048

openssl req -new -key harish.key -out harish.csr -subj "/CN=harish"

cat harish.csr | base64 | tr -d '\n'

kubectl apply -f csr.yml

kubectl get csr

kubectl certificate approve harish-csr


kubectl certificate deny harish-csr

openssl x509 -req -in harish.csr -CA 
openssl x509 --noout  -dates -in harish.crt