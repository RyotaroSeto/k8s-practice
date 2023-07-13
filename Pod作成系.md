### pod作成
- 指定イメージのpod作成
  - kubectl run pod名 --image=イメージ名
- イメージ,ラベル指定でpod作成
  - kubectl run messaging --image=redis:alpine --labels="tier=msg"
- image:redis:alpineでredis-storageというPodを作成し、emptyDir型のVolumeをPodの存続期間だけ使用する
  - kubectl run redis-storage --image=redis:alpine --dry-run=client -o yaml > red.yaml
  - https://kubernetes.io/ja/docs/concepts/storage/volumes/
  - 上記のURLを参考にする
  - 以下正解yaml。volumeMountsを作成しmountPathに指定されたパスを設定volumeMountsとvolumesのnameは同じでなければならない。(名前は同じならなんでも良い。)
```
apiVersion: v1
kind: Pod
metadata:
  name: test-ebs
spec:
  containers:
  - image: registry.k8s.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /test-ebs
      name: test-volume
  volumes:
  - name: test-volume
```
