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
