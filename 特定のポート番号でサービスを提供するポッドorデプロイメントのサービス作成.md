### 特定のポート番号でサービスを提供するポッドのサービス作成
- kubectl expose pod pod名 --port=ポート番号 --name=サービス名

### 特定のノードポート番号でサービスを提供するデプロイメントのサービス作成。またwebアプリのportにlistenする
- kubectl expose deploy hr-web-app  --name=hr-web-app-service --type NodePort --port 8080
- サービスの中のnodeport番号編集　kubectl edit svc hr-web-app-service

https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#expose
