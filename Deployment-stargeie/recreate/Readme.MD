kubectl run test-nginx \
  --image=curlimages/curl \
  --rm -it \
  --restart=Never \
  -- sh -c 'while true; do curl -s http://nginx-svc.default.svc.cluster.local:80; echo; sleep 1; done'


kubectl rollout undo deployment/nginx-deployment