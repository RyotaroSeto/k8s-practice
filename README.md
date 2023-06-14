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

