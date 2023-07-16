### デプロイメントのpod数をスケールさせる

1. kubectl scale deploy nginx-deploy --replicas=3
これでも治らない場合はコントローラーマネージャーを見る
ls /etc/kubernetes/manifests/
