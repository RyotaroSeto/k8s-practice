### etcdのバックアップをとり、指定のファイルに保存



1. etcdの中に入りバックアップ場所に必要な情報を取得する
  - cat /etc/kubernetes/manifests/etcd.yaml
    - endpoints,trusted-ca-file,cert-file,key-fileを探す。
2. バックアップを取る

ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
  --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> \
  snapshot save <backup-file-location>


https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/
