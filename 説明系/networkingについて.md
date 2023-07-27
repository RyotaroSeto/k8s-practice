# networkingについて

## Ingress
Ingressはクラスター外からクラスター内ServiceへのHTTPとHTTPSのルートを公開する
- L7ロードバランサー
  - レイヤー4のロードバランサーに比べて、はるかに高度な方法でネットワークトラフィックをルーティング
  - レイヤー7ロードバランシングは、パケットベースのレイヤー4ロードバランシングに比べるとCPUへの負荷が大きくなりますが、最新サーバでパフォーマンスの低下を引き起こすことはほとんどありません。
  - コンテンツに最適化や変更（圧縮、暗号化など）を適用でき、バッファリングを利用し、低速の接続を上流サーバからオフロードすることで、パフォーマンスを向上できる

## CoreDNS
- Linux ホストを DNS サーバとして構成するためのもの
- 実行可能バイナリとしても、Docker コンテナとしても構成できる
- CoreDNS は Corefile ファイルから構成を読み取る。例えば、ここに `hosts /etc/hosts` と書くとホストの `/etc/hosts` ファイルから DNS を構成できる

## Network Namespaces
Linuxホストで新しいネットワークスペースを作成する
- ip netns add <namespace>

ネームスペース内でコマンドを実行
- ip netns exec <namespace> <command>

ns 間の直接接続は virtual cable(pipe) で行える
- インターフェースの作成
  - ip link add <veth-a> type veth peer name <veth-b>
- ネームスペースへの割り当て
  - ip link set <veth-a> netns <namespace-a>
- IP アドレスの設定
  - ip -n <namespace-a> addr add x.x.x.x dev <veth-a>
- インターフェースの起動
  - ip -n <namespace-a> link set <veth-a> up

linux bridge
- ブリッジの作成
  - ip link add <v-net-n> type bridge
- ブリッジの起動
  - ip link set dev <v-net-n> up
- インターフェースの作成
  - ip link add <veth-a> type veth peer name <veth-a-bridge>
- インターフェースの割り当て
  - ip link set <veth-a> netns red
- ブリッジへの接続
  - ip link set <veth-a-bridge> master <v-net-n>
- IPアドレスの割り当て
  - ip -n <namespace-a> addr add x.x.x.x dev <veth-a>
- 起動
  - ip -n <namespace-a> link set <veth-a> up

## Docker Networking
- None network(ネットワークインターフェースをアタッチしない)
  - docker run --network none <image>
- ホストのネットワークを利用する（ネットワーク的に独立しない）
  - docker run --network host <image>
- コンテナ毎にIPアドレスが払い出され、バーチャルブリッジが作成されてネットワークとして独立する（デフォルト）
  - docker run <image>
- ネットワークモードを確認
  - docker network ls

## CNI in Kubernetes
- k8s では、kubelet が CNI plugin を呼び出す
- kubelet の --network-plugin, --cni-bin-dir, --cni-conf-dir で指定される
  - ps -aux | grep kubelet で確認できる

## Service Networking
- Service は cluster-wide に作成され、いかなるノードに作成された Pod も Service を介して Pod にアクセスできる
- Pod では、namespace や interface を作成して IP アドレスを割り当てていたが、Service の場合はそうではなく、namespace, interface は作成されない。あくまで仮想的なもの。
- Service が作成されると、規定の IP レンジから IP が払い出され、kube-proxy が forwarding rule を作成する

## CoreDNS in Kubernetes
- CoreDNS は ReplicaSet として kube-system にデプロイされる
- CoreDNS は /etc/Corefile で設定され、ConfigMap として挿入される
- 様々なプラグインを利用でき、kubernetes plugin もある
- CoreDNS が構築されると、この Pod に接続するための Service も作成され、これが Node の resolv.conf に nameserver として設定される
  - resolv.conf への設定は kubelet が行い、config.yaml の clusterDNS に設定する
