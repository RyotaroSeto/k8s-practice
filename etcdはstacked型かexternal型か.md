### etcdはstacked型かexternal型か
1. kubectl describe podでpodの中身確認

--etcd-server=https://127.0.0.1:2379
でローカルホストのIPを指しているためコントロールプレーンノード自体で動作。なのでstacked型


etcdのpodが存在しない場合。ssh でnodeにアクセス
cat /etc/kubernetes/manifests/etcd.yaml ない場合

kube-apiserver-cluster2-controlplane　podの中身確認

--etcd-servers=https://192.16.199.19:2379のため外部。external型
