### etcd-serverが属するETCDクラスタのノードの数は？

ETCDCTL_API=3 etcdctl \
>  --endpoints=https://127.0.0.1:2379 \
>  --cacert=/etc/etcd/pki/ca.pem \
>  --cert=/etc/etcd/pki/etcd.pem \
>  --key=/etc/etcd/pki/etcd-key.pem \
>   member list


https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/#backing-up-an-etcd-cluster
