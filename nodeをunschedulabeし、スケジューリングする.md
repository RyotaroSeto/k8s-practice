### ワーカーノード node01 を Unschedulable にする。これが完了したら、dev-redis というポッドを作成し、redis:alpine というイメージで、このワーカーノードにワークロードがスケジューリングされないようにします。最後に、prod-redis という新しいポッドを作成し、redis:alpine というイメージで、node01 にスケジューリングされるようにします。

1. kubectl taint nodes node01 env_type=production:NoSchedule
2. kubectl run dev-redis --image=redis:alpine
3. kubectl run prod-redis --image=redis:alpine --dry-run=client -o yaml > r.yamlで作成し以下を追加する
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"


https://kubernetes.io/ja/docs/concepts/scheduling-eviction/taint-and-toleration/
