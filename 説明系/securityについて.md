# securityについて
全てのユーザーアクセスは`kube-apiserver`で管理されており、kubectlなどの全てのリクエストは`kube-apiserver`を経由している。`kube-apiserver`はリクエストを処理する前に認証する。

## どのように`kube-apiserver`はどのように認証しているのか?
- ユーザーとパスワードのリストをCSVファイルで作成し、それをユーザー情報のソースとして使用。そしてそのファイルをオプションとして`kube-apiserver`に渡す。
  - 具体的に`kube-apiserver`の設定ファイルのExecStartかcommandに`--basic-auth-file=CSV名`を追加(大体は`/etc/kubernetes/manifests/kube-apiserver.yaml`)
  - これらのユーザーに必要なロールとロールバインディングを作成する
  - 作成後、ユーザー認証情報を使用してkube-apiサーバーに認証できます。
  - `curl -v -k https://localhost:6443/api/v1/pods -u "user1:password123"`
```
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

---
# This role binding allows "jane" to read pods in the "default" namespace.
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: read-pods
  namespace: default
subjects:
- kind: User
  name: user1 # Name is case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role #this must be Role or ClusterRole
  name: pod-reader # this must match the name of the Role or ClusterRole you wish to bind to
  apiGroup: rbac.authorization.k8s.io
```

## TLSについて

### open SSLコマンドを使用して、秘密鍵と公開鍵のペアを生成
- `openssl genrsa -out my-bank.key 1024`
  - my-bank.key
- `openssl rsa -in my-bank.key -pubout > mybank.pem`
  - my-bank.key mybank.pem
