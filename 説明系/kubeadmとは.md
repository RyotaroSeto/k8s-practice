# kubeadmについて

クラスター環境を高速に作成する方法として`kubeadm init`と`kubeadm join`というコマンドを提供するツール。Kubernetesのクラスターを立ち上げ、稼働させる上で最低限必要なものを提供

Kubernetesをシンプルに構築できるようにするため、kubeadmは限られた定数・変数、そしてファイルしか使用しない。
ディレクトリ構造、使用するファイルは以下。

- /etc/kubernetes: アプリケーションの中心となるディレクトリ
  - kubelet.conf
  - controller-manager.conf
  - scheduler.conf
  - admin.conf: クラスター認証、kubeadmのために利用する
- /etc/kubernetes/manifests: kubeletが静的なPodマニフェストを探す際の場所
  - etcd.yaml
  - kube-apiserver.yaml
  - kube-controller-manager.yaml
  - kube-scheduler.yaml
- 各種証明書とキーファイル
  - ca.crt,ca.key: Kubernetes Certificate Authority(認証局)
  - apiserver.crt,apiserver.key: APIサーバ認証書とキーファイル
  - apiserver-kubelet-client.crt,apiserver-kubelet-client.key: APIサーバで使われるクライアント認証書とキーファイル。これを使ってkubeletと安全に接続する
  - sa.pub, sa.key: ServiceAccountに署名するときにcontroller managerで使われるキーファイル
  - front-proxy-ca.crt, front-proxy-ca.key:front proxy認証局
  - front-proxy-client.crt, front-proxy-client.key: front proxyクライアントの証明書とキーファイル


## kubeadm init
何をしているか全体の流れは以下。
- init前のチェック(preflight checks)
- kubelet起動
- 各種証明書の作成
- control plane用のファイル作成(kubeconfig, manifest)
- etcd用のファイル作成(manifest)
- control planeの起動待ち
- kubeadm,kubeletのコンフィグアップロード
- control planeラベリング
- TLS Bootstrap
- addonsインストール(kube-proxy, DNS)


## kubeadm join
kubeadmでクラスターを開始する際、双方向でのtrustが必要になる。ここではdiscoveryとTLS bootstrapの2つのステップに分かれる。discoveryでは、API serverのIPアドレスとkubeconfigファイルを利用するために探索するステップになります。ユーザが指定することもできる。全体の流れは以下
- init前のチェック
- cluster-infoのdiscovery
- TLS Bootstrap


## kubeadm install手順
1. このURLにある以下を実行　https://kubernetes.io/ja/docs/setup/production-environment/container-runtimes/

```
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter

# この構成に必要なカーネルパラメーター、再起動しても値は永続します
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF

# 再起動せずにカーネルパラメーターを適用
sudo sysctl --system
```

2. `cat /etc/*-release`でインストール対象OS確認

3. このURLにある手順を実行しインストールする https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/install-kubeadm/

1. sudo apt-get update
2. sudo apt-get install -y apt-transport-https ca-certificates curl
3. curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg
4. echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list
5. sudo apt-get update
6. sudo apt-get install -y kubelet=1.27.0-00 kubeadm=1.27.0-00 kubectl=1.27.0-00
7. sudo apt-mark hold kubelet=1.27.0-00 kubeadm=1.27.0-00 kubectl=1.27.0-00
