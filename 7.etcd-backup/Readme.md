etcd backup

https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/

cat /etc/kubernetes/manifests/etcd.yaml

ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  member list

ETCDCTL_API=3 etcdctl --endpoints 127.0.0.1:2379 \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  snapshot save /opt/etcd-snapshot.db

etcdctl --write-out=table snapshot status /opt/etcd-snapshot.db
  

# Restore

ETCDCTL_API=3 etcdctl --data-dir /var/lib/etcd-from-backup snapshot restore /opt/etcd-snapshot.db

vi /etc/kubernetes/manifests/etcd.yaml
:se nu

sed -i 's/var/lib/etcd: /var/lib/etcd-from-backup: /g' /etc/kubernetes/manifests/etcd.yaml

watch "crictl ps | grep etcd"