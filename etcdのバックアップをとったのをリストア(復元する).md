### etcdのバックアップをとったのをリストア(復元する)

1. ETCDCTL_API=3 etcdctl snapshot restore --data-dir /var/lib/etcd-from-backup /opt/snapshot-pre-boot.db


2. vi /etc/kubernetes/manifests/etcd.yamlでhostPath:をバックアップで復元したディレクトリを指定する。上でいう、/var/lib/etcd-from-backup

https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/
