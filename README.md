# k8sコマンド
- ヘルプ確認
  - kubectl run --help
- nginxのイメージpod作成
  - kubectl run nginx --image=nginx
- pod作成
  - kubectl create -f pod-definition.yaml (createの代わりにapplyでもOK)
- pod確認
  - kubectl get pods
- pod詳細確認
  - kubectl describe pod myapp-pod
- pod削除
  - kubectl delete pod myapp-pod
- pod確認(詳細に) nodeの配置場所などわかる
  - kubectl get pods -o wide
- nginxのpod作成
  - kubectl get pods -o wide
- redisのpod(imageがredis123)のマニフェスト確認
  - kubectl run redis --image=redis123 --dry-run -o yaml
  - dry-run=client フラグを使用すると、実際に送信することなく、クラスターに送信されるオブジェクトを確認することができます。
- redisのpod(imageがredis123)のマニフェスト作成
kubectl run redis --image=redis123 --dry-run=client -o yaml > redis.yaml
- レプリケーションコントローラー確認
  - kubectl get replicationController
- レプリケーションコントローラー削除
  - kubectl delete replicationController myapp-rc
- レプリカセット確認
  - kubectl get replicaset
- レプリカセット削除
  - kubectl delete replicaset myapp-replicaset
- マニフェストのレプリカセットの数を変更した時更新
  - kubectl replace -f replicaset-definition.yaml
- コマンドでレプリカセットの数を変更したい時
  - kubectl scale --replicas=6 replicaset-definition.yaml(kubectl scale --replicas=6 replicaset myapp-replicaset)
- デプロイメント確認
  - kubectl get deployments
- 作成されたもの全て確認
  - kubectl get pods
- 特定の名前とイメージを持つポッドやデプロイメントを作成するよう求められた場合
  - kubectl run コマンド
- deploymentのヘルプ
  - kubectl create deployment --help
- deployment作成
  - kubectl create deployment httpd-frontend --image=httpd:2.4-alpine --replicas=3
- service
 - NodePort
 - ClusterIP
 - Loadbalancer
   - クラウドプロバイダー(awsなど)のロードバランサーを使用して、Serviceを外部に公開します。クラスター外部にあるロードバランサーが転送する先のNodePortとClusterIP Serviceは自動的に作成されます。
 - serviceのClusterIPはKubernetesのクラスター外からはアクセスができない。serviceのNodePortを利用するとNodeの特定ポートにアクセスすれば、ClusterIPに繋げてくれる。
- namespace表示
  - kubectl get pods --namespace=kube-system
- namespace作成
  - kubectl create namespace dev
  - kubectl apply -f namespace-definition.yaml
- namespace付きpod作成
  - kubectl create -f pod-definition.yaml --namespace=dev
  - metadataの中にnamespace: devつける
- podの中にある全てのnamespaceを見る
  - kubectl get pods --all-namespaces(kubectl get pods -A)
- リソース定義をYAML形式で画面上に出力します。
  - -o yaml
  - kubectl run nginx --image=nginx --dry-run=client -o yaml
  - kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml
- NodePort タイプの nginx という名前の Service を作成して、ノード上のポート 30080 に pod nginx のポート 80 を公開
  - kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml
  - kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml
- Pod Name: redis,Image: redis:alpine,Labels: tier=db
  - kubectl run redis --image=redis:alpine --labels="tier=db"
- service clusterip のヘルプ
  - kubectl create service clusterip --help
- nameがhttpd,imageがhttpd:alpine,
  - kubectl run httpd --image=httpd:alpine --port=80 --expose=true
- スケジューラーの有無
  - kubectl get pods --namespace kube-system　でスケジューラが見つかる`scheduler`と書いてある。なければpodがpendingのままの状態になる
- label指定のget pods
  - kubectl get pods --selector name=myapp
- ヘッダー抜きで指定のラベルのpodの数を表示
  - kubectl get pods --selector env=dev --no-headers | wc -l
- podやreplicasetなど全てのラベルの数を表示
  - kubectl get all --selector env=prod --no-headers | wc -l
- 複数ラベル指定
  - kubectl get all --selector env=prod,bu=finance,tier=frontend
- アプリケーションbleuのpodにノードを専用にする key=value app=blue
  - kubectl taint nodes node-name key=value:taint-effect
- imageがnginxでmosquitoという名のpod作成
  - kubectl run mosquito --image=nginx
- yamlに転記
  - kubectl run bee --image=nginx --dry-run=client -o yaml > bee.yaml
- すでに指定のnodeにtaintが設定してあって削除したい場合
  - kubectl taint node controlplane node-role.kubernetes.io/control-plane:NoSchedule-  #最後がtaint名「-」をつけると削除される
- nodeにlabelをつける これでpodにnodeSelectorのsizeを対象のlabel名で指定できる
  - kubectl label nodes <node-name> <label-key>=<label-value>
- ↑上記は複雑な場合目的を果たすことが難しい。そのためノードアフィニティ(Node Affinity)とアンチアフィニティがある
  - Node Affinityの目的はpodが特定のノードにホストされるようにする
    - 特定のノードへのポッド配置を制限する高度な機能
- 指定のノードに何か配置していないか調べる。配置されていない場合はnone
  - kubectl describe node node01 | grep Taints # Taints <none>
  - kubectl describe node controlplane | grep Taints # Taints <none>
- 現状のpodをyamlに移してくれる
  - kubectl get pod elephant -o yaml > elephant.yaml
-  指定のネームスペースにあるデーモンセットの詳細確認
  - kubectl get daemonsets -A # deamonsetの詳細確認
  - kubectl describe daemonsets kube-proxy -n kube-system
- DeamonSetはデプロイメントの特殊系
- DaemonSetのユースケースとしては各Podが出すログを収集するFluentdや、各Podのリソース使用状況やノードの状態をモニタリングするDatadogなど全Node上で必ず動作している必要のあるプロセスのために利用
- 静的pod定義ファイルを格納するディレクトリのパス
  - cat /var/lib/kubelet/config.yaml
- kube-api serverの静的podとしてdeployするのに必要なDockerイメージはどのようなものか
  - cat /etc/kubernetes/manifests/kube-apiserver.yaml
- busyboxイメージとsleep 1000コマンドを使用するstatic-busyboxという名前の静的ポッドを作成
  - kubectl run static-busybox --image=busybox --restart=Never --dry-run=client -o yaml --command -- sleep 1000 > test-box.yaml
  - cp test-box.yaml /etc/kubernetes/manifests/
- 現在のnamespaceにある全てのeventをリストアップしスケジュールされたeventを探す
  - kubectl get events -o wide
- namespaceにあるpodの詳細を見る時はnamespaceを指定する
  - kubectl describe pod kube-scheduler-controlplane -n kube-system
- 各ノードのCPUとメモリの消費量を表示
  - kubectl top node
- podのパフォーマンスメトリクス
  - kubectl top pod
- podのログ表示
  - kubectl logs -f event-pod
- podの中に複数コンテナがある場合のログ表示 podの後にコンテナ名もしていする　
  - kubectl logs -f event-pod event-simulator
- ロールアウトのステータス確認
  - kubectl rollout status deployment/myapp-deployment
- デプロイのリビジョンと履歴の確認
  - kubectl rollout history deployment/myapp-deployment
- デプロイメントは以前のリビジョンにロールバックすることができる
  - kubectl rollout undo deployment/myapp-deployment
- Command line arguments: --color=greenのpod作成
  - kubectl run webapp-green --image=kodekloud/webapp-color -- --color=green
- コマンドラインでconfigマップ作成する方法
  - kubectl create configmap app-config --from-literal=APP_COLOR=blue --from-literal=APP_MODE=prod
- コンフィグマップ表示
  - kubectl get configmaps
- コマンドラインでsecret作成する方法
  - kubectl create secret generic app-secret --from-literal=DB_HOST=mysql
- base64でエンコードHash化
  - echo -n "mysql" | base64
- secret表示
  - kubectl get secrets
- 既存のsecretのyaml表示
  - kubectl get secret app-server -o yaml
- 既存のpodなどを強制的に差し替え
  - kubectl replace --force -f /tmp/kubectl-edit-2804231204.yaml
- serviceのログ
  - kubectl logs orange -c init-myservice
- nodeを切り離す(指定したノード上の Pod を安全に削除して,そのノードには新しく Pod をスケジューリングしないようにステータスを変更できる)
  - kubectl drain node-1
-  デーモンセットが載っている時切り離す場合
  - kubectl drain node-1 --ignore-daemonsets
- 強制的に切り離す
  - kubectl drain node-1 --ignore-daemonsets --force
- ノードを復帰させる
  - kubectl uncordon node-1
- 余分なポッドがノードに追加されるのを防ぐ.ノードが cordon された後は、新しいポッドをそのノードにスケジュールすることはできない。
  - kubectl cordon node-2
- kubernetesクラスタのアップグレード
  - このクラスタでワークロードをホストできるノードの数は？
    - kubectl describe node | grep Taints
  - 現在のkubernetesの最新version確認
    - kubeadm upgrade plan
  1. kubectl get nodes     controlplane,node01アリ
  2. 最初にコントロールプレーンノードをアップグレードするために、コントロールプレーンノードからワークロードを排出し、スケジューリング不能にする。
    - kubectl drain controlplane --ignore-daemonsets
  3. コントロールプレーンをアップグレードする(kubeadmツールをアップグレードし,次にcontrolplaneコンポーネントをアップグレードし、最後にkubeletをアップグレードする。)
    1. osを確認
      - cat /etc/*release*
    2. osに準拠してupdate
      - apt update
      - apt-cache madison kubeadm
    3. コントロールプレーンをupdate
      -  apt-mark unhold kubeadm && \
         apt-get update && apt-get install -y kubeadm=1.27.0-00 && \
         apt-mark hold kubeadm
    4. kubeadmのversion確認
      - kubeadm version
    5. 今回のアップグレードで選択したパッチのバージョンに置き換える。
      - sudo kubeadm upgrade apply v1.27.0
      - apt-mark unhold kubelet kubectl &&\
        apt-get update && apt-get install -y kubelet=1.27.0-00 kubectl=1.27.0-00 &&\
        apt-mark hold kubelet kubectl
    6. 再起動
      - sudo systemctl daemon-reload
      - sudo systemctl restart kubelet
    7. ノードを復帰させる
      - kubectl uncordon controlplane
  4. node01をアップグレードする
    1. 以下node01も切り離す
      - kubectl drain node-1 --ignore-daemonsets
    2. nodeのINTERNAL-IP取得,ログイン
      - kubectl get nodes -o wide
      - ssh 192.25.175.3
    3. kubeadminをupgrade
      -  apt-mark unhold kubeadm && \
         apt-get update && apt-get install -y kubeadm=1.27.0-00 && \
         apt-mark hold kubeadm
      - sudo kubeadm upgrade node
      - apt-mark unhold kubelet kubectl && \
        apt-get update && apt-get install -y kubelet=1.27.0-00 kubectl=1.27.0-00 && \
        apt-mark hold kubelet kubectl
      - sudo systemctl daemon-reload
      - sudo systemctl restart kubelet
    4. ノードを復帰させる
      - kubectl uncordon node01
  - etcdのスナップショットを取りたい場合
    - export ETCDCTL_API=3
    - ETCDCTL_API=3 etcdctl --endpoints=https://[127.0.0.1]:2379 \
      > --cacert=/etc/kubernetes/pki/etcd/ca.crt \
      > --cert=/etc/kubernetes/pki/etcd/server.crt \
      > --key=/etc/kubernetes/pki/etcd/server.key \
      > snapshot save /opt/snapshot-pre-boot.db
    - snapshot save /opt/snapshot-pre-boot.db
  - etcdはTLS-Enabledなので、以下のオプションは必須
   - -cacert
     - CAバンドルを使用するTLS対応セキュア・サーバーの証明書を検証する。
   - -cert
     - TLS証明書ファイルを使ってセキュアなクライアントを識別
   - --endpoints=[127.0.0.1:2379]
     - etcdがマスターノード上で動作しており、localhost 2379上で公開されているため、デフォルト
   - --key
     - TLSキーファイルを使用するセキュアなクライアントを識別
  - スナップショットリストアのhelpオプション表示
    - etcdctl snapshot restore -h
  - アプリで使用しているetcdのversion
    - kubectl get pods -n kube-system
    - kubectl describe pod etcd-controlplane -n kube-system
  - バックアップファイルを使ってクラスタの元の状態を復元
    - ETCDCTL_API=3 etcdctl snapshot restore --data-dir <data-dir-location> snapshotdb
    - /var/lib/etcd/ を/var/lib/etcd-from-backupにしているから
    - cat /etc/kubernetes/manifests/etcd.yaml のvolumes:　hostPath:を修正する
  - Stacked etcd topology
    - スタック化されたコントロールプレーンノードの内部にそれぞれ etcd を配置してクラスターを構成
  - External etcd topology
    - コントロールプレーンの外部に etcd クラスターを構成
  - デフォルトで作成される3つのNamespace
    - default … Namespaceを持たないオブジェクトのデフォルトのNamespace
    - kube-system … Kubernetesによって作成されたオブジェクトのためのなNamespace
    - kube-public … 全ユーザーが読み込めるNamespace

## Security
- apiで認証情報ファイル付きでアクセス
  - ExecStart=/usr/local/bin/kube-apiserver --base-auth-file=user-details.csv
- kubeapiserverのyamlに認証情報加える
  - /etc/kubernetes/manifests/kube-apiserver.yaml   のcommandの中に --base-auth-file=user-details.csvを加える
- サービスアカウントの作成(CKADの内容)
  - kubectl create serviceaccount sa1
- サービスアカウント取得(CKADの内容)
  - kubectl get serviceaccount
- クラスタの証明書生成
  - 証明書署名要求(お客様の詳細情報を全て記載記載した証明書)生成
    - openssl genrsa -out ca.key 2048
    - openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
    - openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
  - 管理者ユーザーの証明書生成
    - openssl genrsa -out admin.key 2048
    - openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr
    - openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
  - apiserver証明書生成
    - openssl genrsa -out apiserver.key 2048
    - openssl req -new -key apiserver.key -sub "/CN=kube-apiserver" -out apiserver.csr
    - openssl x509 -req -in apiserver.csr -CA ca.crt -CAkey ca.key -out apiserver.crt
- クラスタ全体の全証明書のヘルスチェック
  - cat /etc/kubernetest/manifests/kube-apiserver.service
  - cat /etc/kubernetest/manifests/kube-apiserver.yaml
  - 各証明書の中身確認(証明書の名前(CN Name)、別名(alt Names)、組織(organization)、発行者名(issuer)、有効期限(expiration))
    - openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
- `kube-api` serverに使用されている証明書ファイルを特定
  - cat でkube-apiのマニフェスト確認
- 新しいメンバーがjoinして証明書署名があり、それを使用して、チームのクラスタにアクセスできるようにする手順
  - Create a CertificateSigningRequest
    - 中身kind: CertificateSigningRequestのyaml作成
    - ディレクトリにあるakshay.csrを`cat akshay.csr | base64 -w 0`で生成したもおんをやyamlのrequestに入れる
    - kubectl apply -f akshay.yaml
  - 作成したcertificateの状態確認
    - kubectl get csr
  - まだpendingなので承認する
    - kubectl certificate approve akshay
  - csr拒否する
    - kubectl certificate deny agent-smith
  - 拒否したのを削除する
    - kubectl delete csr agent-smith
- 今までkubectlコマンドで認証情報を記載して叩いていたがそれは面倒なのでyamlで定義する。それがkubeConfig
  - $HOME/.kube/configにconfigファイルを置いて認証されたpodをみる
    - kubectl get pods
  - kubeconfigの設定ファイル確認
    - kubectl config view
  - context変更
    - kubectl config use-context　XXXXXX
    - kubectl config use-context コンテキスト名 --kubeconfig コンフィグ名
    - kubectl config use-context research --kubeconfig /root/my-kube-config
  - role表示
    - kubectl get roles
    - kubectl describe role XXX
  - rolebinding表示
    - kubectl get rolebindings
    - kubectl describe rolebinding XXX
  - ユーザーに対して必要な権限作成
    - kubectl create role developer --namespace=default --verb=list,create,delete --resource=pods
    - kubectl create rolebinding dev-user-binding --namespace=default --role=developer --user=dev-user
    - or
```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["list", "create","delete"]

---
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: dev-user-binding
subjects:
- kind: User
  name: dev-user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```
  - role編集
    - kubectl edit role ロール名 -n blue
  -  指定のユーザーで実行
    - --as dev-user
- nodeや永続volumeなどクラスタ全体のリソースに対して、ユーザ認可方法
  - クラスタロールとクラスタロールバインディング使用
  - クラスタロール
    - クラスタ管理者にクラスタ内のノード表示、作成、削除を与えるクラスタ管理者ロールを作成できる
    - ストレージ管理者のロールを作成して、ストレージ管理者に永続volumeとclaimを作成する権限を与えれる
- サービスアカウント(CKAD)
  - 認証、認可、ロールベースのアクセス制御などkubernetesの他のセキュリティ関連の概念と連携
  - kubernetesにはユーザーアカウントとサービスアカウント、2種類のアカウントがある
  - ユーザーアカウントは人間が使うもの
  - サービスアカウントは機械が使うもの
  - サービスアカウント作成
    - kubectl create serviceaccount XXX
  - サービスアカウント表示
    - kubectl get serviceaccount
    - kubectl describe serviceaccount XX
  - secretトークンは/var/run/secret/kubernetes
    - kubectl exec -it XXXX --ls /var/run/secret/kubernetes.io/serviceaccount
- プライベートイメージ
  - 既存のimageをprivateにしたい場合,nginx:alpine 　→ myprivateregistry.com:5000/nginx:alpine
- podがどのユーザーで起動しているか確認するコマンド
  - kubectl exec pod名 -- whoami
- ネットワークポリシー
  - 指定のpodからの何番portのみのアクセスのみ許可するなどの設定
  - 具体的にはpodにlabelをつけ、ネットワークポリシーのportセレクタフィールドに同じlabelをつける
  - Ingress(受信)
    - 例.DBpodからAPIpodからの受信トラフィックを許可すること。
    - podSelector
      - ラベルでpodを選択する
    - namespaceSelector
      - ラベルで名前空間を選択する
    - ipBlock
      - IPアドレス範囲を選択する
  - Egress(送信)
    - 例.バックアップサーバーがバックアップを開始する代わりにdb-pod上のバックアップをバックアップサーバーにpushするエージェントがあるとする
    - この場合、トラフィックはdb-podから外部のバックアップサーバー。
  - ネットワークポリシー取得
    - kubectl get networkpolicy
- Storage(永続volume)
  - volume
    - コンテナは削除作成が頻繁に行われるがコンテナ内のデータを永続化するためのもの
  - Persistent Volume
    - クラスタ上でアプリケーションを展開するユーザが使用できるように、管理者が設定したクラスタ全体のストレージボリュームのプール
    - このプールから永続的なvolumeを指定してストレージを選択する
    - kubectl get persistentvolume
  - Persistent Volume Claim
    - ユーザーによって要求されるストレージ。Podと似ていてPodはNodeリソースを消費し、PVCはPVリソースを消費。
    - クレームは特定のサイズやアクセスモード(例えば、ReadWriteOnce、ReadOnlyMany、ReadWriteManyにマウントできる)
    - kubectl get persistentvolumeclaim
    - kubectl delete persistentvolumeclaim XXX
  - pvとpvcのアクセスモードが一致しないと連携されない
- ネットワーク
  - ip link(再起動したら消える)
    - ホスト上のインターフェイスをリストアップし、変更すること。
  - ip addr(再起動したら消える)
    - インターフェイスに割り当てられているIPアドレスを確認すること
  - ip addr add 192.168.XXXX dev eth0(再起動したら消える)
    - インターフェイスにIPアドレスを設定するために使用
  - cat /proc/sys/net/ipv4/ip_forward
- DNS
  - エントリ追加(IPアドレスに名前つける)       192.168.XXXX  db
    - cat >> /etc/hosts
  - 各　/etc/hostsにあったら増えたら一気に修正することが困難なため一元的に管理するため**DNSサーバー**が生まれた.
  - 全てのhostをDNSサーバーの情報を参照する
  - 全てのホストは/etc/resolv.confにDNS解決の設定ファイルを持っている
    - cat /etc/resolv.conf
  - ネットワーク・ネームスペースをテストしている間に、一方のネームスペースからもう一方のネームスペースにpingを送信できない問題が発生した場合IPアドレスを設定する際にNETMASKを設定。
    - ip -n red addr add 192.168.1.10/24 dev veth-red
  - bridgeインターフェイスを表示
    - ip address show type bridge
  - bridgeインターフェイスのIPアドレスを設定
    - ip addr add 10.244.1.1/24 dev XXX
  - bridgeネットワーク作成
    - ip link add XXXX type bridge
  - bridgeネットワークset
    - ip link set XXXX type bridge
  - ip route調べる
    - ip route
  - どのportでlistenしているか調べる
    - netstat -npl
- CNL Kubernetes
  - kubeletを検査し、コンテナランタイムのエンドポイント値がKubernetes用に設定されていることを確認
    - ps aux | grep kubelet | grep runtime
  - kubernetesクラスタで使用するCNIプラグインはどのように設定されているか?
    - ls /etc/cni/net.d
  - weave
    1. wget https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
  - 使用するプラングインの種類、サブネット、ルート表示
    - cat /etc/cni/net.d/XXXX.conf
- ClusterIP
  - 特定のnodeに縛られることはないが、そのサービスにはクラスタ内からしかアクセスすることができないサービス
- Serviceはpodとは異なり、各nodeで作成されたり、各nodeに割り割り当てられたりすることはない。serviceはクラスタ全体の概念
- nodeのIPアドレス探し方
  - ip addで特定のIPアドレスを持っているインターフェイスを探す。ここではeth0
  - ipcalcツールを使ってネットワークの詳細を見る.ipcalc -b 192.3.8.255
- CoreDNS
  - kubernetesのDNS設定ファイル場所は、/etc/coredns
  - コアファイルはconfig mapオブジェクトとしてpodに渡されるためkubectl get configmap -n kube-systemで表示
- Ingress
  - 以下のようにIngressを設定しなければならない。ユーザーが左のURLにアクセスすると、そのリクエストは内部的に右のURLに転送される
    - `http://<ingress-service>:<ingress-port>/watch --> http://<watch-service>:<port>/`
    - `http://<ingress-service>:<ingress-port>/wear --> http://<wear-service>:<port>/`
  - rewrite-targetオプションがなければ、こうなる**重要**
    - `http://<ingress-service>:<ingress-port>/watch --> http://<watch-service>:<port>/watch`
    - `http://<ingress-service>:<ingress-port>/wear --> http://<wear-service>:<port>/wear`
  - 外部のロードバランサーを使用する場合はGKEとかEKS。
  - 内部のロードバランサーを使用する場合、nginx-ingress-controllerをdeployしなければならない。構成は以下。上から順番
    - LoadBalancer Service(ingress-endpoint)
      - Deployment(nginx-ingress-controller)
        - service(service-a) →　Deployment(service-a)
        - service(service-b) →　Deployment(service-b)
        - service(service-c) →　Deployment(service-c)
- HAモードでのetcd設定
  - etcdはraftプロコトルを用いて分散コンセンサスを実装している
  - HA環境の最小ノード数は３台である(多数決を採用しており、2台は1台と一緒のため。なので偶数は対象外)
- トラブルシューティング
  - Webアプリケーションの場合
    1. curlを使用してノードポートのIPでWebサーバーにアクセスできるか確認する。
      - curl http://web-service-ip:node-por
    2. serviceの確認
      - web Podのエンドポイントを確認できるか
      - kubectl describe service web-service
      - `エンドポイント``port`が正しい場合は、サービスに設定されているセレクタとポッドに設定されているセレクタを比較する。必ず一致させること。
    3. pod本体の確認
      - 実行状態であることを確認
      - get pod, describe pod, logs
      - 失敗している場合ログを確認。kubectl logs web
    4. DB serviceの状態確認
      - セレクタ,service名の確認。podまたはdeploymentに記載されているDB_Host,DB_User,DB_Password:がserviceのものと合っているか
    5. DB pod本体確認
  - コントロールプレーンの場合
    1. kubectl get nodes でクラスタ内のノードの状態を確認する。
    2. kubectl get pods でクラスタ上のポッドの状態を確認する。
      - kubectl get pods -n kube-system
      - kube-system内のpodに問題がある場合、`ls /etc/kubernetes/manifests/`内を修正.  edit pods　はできない。
    3. kubeadmでデプロイされた場合、コントロールプレーンコンポーネントをポッドとしてdeployしていれば、kube-systemの名前空間でpodsが動作しているか確認
    4. コントロールプレーンプレーンのコンポーネントがサービスとしてデプロイされている場合、
      - service kube-apiserver status
      - service kube-controller-manager status
      - service kube-schduler status
      - service kubelet status
      - service kube-proxy status
    5. それでも解決しない場合、コントロールプレーンのコンポーネントのログを確認
      - kubectl logs kube-apiserver-master -n kube-system
  - ワーカーノードの場合
    1. kubectl get nodes でクラスタ内のノードの状態を確認する。
      - もしおかしい場合describe node でnodeの状態確認
      - 何も問題ない場合、nodeの中入る。ssh node01  確認するのはkubelet. service kubelet status  ノードのコントローラーのため。止まっている場合動かす。service kubelet start
      - それでも治らない場合、journalctl -u kubelet でログ確認できる。ls /etc/kubernetes/manifests/ 周り確認し問題なかったら、 ls /var/lib/kubeletを確認.
      - コントロールプレーンがおかしい場合,portが間違っている可能性あり。正しいportは6443 service kubelet restart
- k8s JSON
  - 下記のJSONでコンテナの順番は異なっても構わず、redisコンテナが常に2番目のコンテナである必要はない場合のredisコンテナのrestartCountを取得する`?(@)`が重要
  - $.status.containerStatuses[?(@.name == 'redis-container')].restartCount
```
{
  "apiVersion": "v1",
  "kind": "Pod",
  "metadata": {
    "name": "nginx-pod",
    "namespace": "default"
  },
  "spec": {
    "containers": [
      {
        "image": "nginx:alpine",
        "name": "nginx"
      },
      {
        "image": "redis:alpine",
        "name": "redis-container"
      }
    ],
    "nodeName": "node01"
  },
  "status": {
    "conditions": [
      {
        "lastProbeTime": null,
        "lastTransitionTime": "2019-06-13T05:34:09Z",
        "status": "True",
        "type": "Initialized"
      },
      {
        "lastProbeTime": null,
        "lastTransitionTime": "2019-06-13T05:34:09Z",
        "status": "True",
        "type": "PodScheduled"
      }
    ],
    "containerStatuses": [
      {
        "image": "nginx:alpine",
        "name": "nginx",
        "ready": false,
        "restartCount": 4,
        "state": {
          "waiting": {
            "reason": "ContainerCreating"
          }
        }
      },
      {
        "image": "redis:alpine",
        "name": "redis-container",
        "ready": false,
        "restartCount": 2,
        "state": {
          "waiting": {
            "reason": "ContainerCreating"
          }
        }
      }
    ],
    "hostIP": "172.17.0.75",
    "phase": "Pending",
    "qosClass": "BestEffort",
    "startTime": "2019-06-13T05:34:09Z"
  }
}
```
  - 下記のJSONでコンテナで、すべてのポッド名を取得
  - $[*].metadata.name
```
[
  {
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
      "name": "web-pod-1",
      "namespace": "default"
    },
    "spec": {
      "containers": [
        {
          "image": "nginx:alpine",
          "name": "nginx"
        }
      ],
      "nodeName": "node01"
    }
  },
  {
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
      "name": "web-pod-2",
      "namespace": "default"
    },
    "spec": {
      "containers": [
        {
          "image": "nginx:alpine",
          "name": "nginx"
        }
      ],
      "nodeName": "node02"
    }
  },
  {
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
      "name": "web-pod-3",
      "namespace": "default"
    },
    "spec": {
      "containers": [
        {
          "image": "nginx:alpine",
          "name": "nginx"
        }
      ],
      "nodeName": "node01"
    }
  },
  {
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
      "name": "web-pod-4",
      "namespace": "default"
    },
    "spec": {
      "containers": [
        {
          "image": "nginx:alpine",
          "name": "nginx"
        }
      ],
      "nodeName": "node01"
    }
  },
  {
    "apiVersion": "v1",
    "kind": "Pod",
    "metadata": {
      "name": "db-pod-1",
      "namespace": "default"
    },
    "spec": {
      "containers": [
        {
          "image": "mysql",
          "name": "mysql"
        }
      ],
      "nodeName": "node01"
    }
  }
]
```
- k8s JSON
  - 下記の配列で以下のように取り出す場合
```
[
  "car",
  "bike"
]
```
  - $[0,3]
```
[
  "car",
  "bus",
  "truck",
  "bike"
]
```
