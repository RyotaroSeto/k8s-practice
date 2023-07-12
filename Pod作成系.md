### pod作成
- 指定イメージのpod作成
  - kubectl run pod名 --image=イメージ名
- イメージ,ラベル指定でpod作成
  - kubectl run messaging --image=redis:alpine --labels="tier=msg"
