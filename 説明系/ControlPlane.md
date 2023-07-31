# Controlplane Services

## service
- service kube-apiserver status
- service kube-controller-manager status
- service kube-scheduler status
- service kubelet status     service kubelet start
- service kube-proxy status

## logs
- kubectl logs kube-apiserver-master -n kube-system
- journalctl -u kube-apiserver
- journalctl -u kubelet

kubeletの設定ファイルの場所
vi /var/lib/kubelet/config.yaml
systemctl restart kubelet
