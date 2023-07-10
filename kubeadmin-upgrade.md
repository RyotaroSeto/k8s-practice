### kubeadminなどのupgrade手順
- 参考URL
  - https://kubernetes.io/ja/docs/setup/production-environment/tools/kubeadm/install-kubeadm/
  - https://kubernetes.io/ja/docs/setup/production-environment/container-runtimes/
  - https://kubernetes.io/docs/tasks/administer-cluster/kubeadm/kubeadm-upgrade/
- 手順
  -  最初にコントロールプレーンノードをアップグレードするために、コントロールプレーンノードからワークロードを排出し、スケジューリング不能にする。
    - kubectl drain controlplane --ignore-daemonsets
  - OS確認
    - cat /etc/*release*
  - OSに準拠してupdate
    - apt update
    - apt-get install kubeadm=1.27.0-00
    - kubeadm upgrade plan v1.27.0
    - kubeadm upgrade apply v1.27.0
    - apt-get install kubelet=1.27.0-00
  - version確認
    - kubeadm version
  - 再起動
    - systemctl daemon-reload
    - systemctl restart kubelet
  - ノードを復帰させる
    - kubectl uncordon controlplane


