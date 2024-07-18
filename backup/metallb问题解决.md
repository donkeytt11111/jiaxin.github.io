添加 Metallb 仓库：

sh
helm repo add metallb https://metallb.github.io/metallb
更新 Helm 仓库：

sh
helm repo update
安装 Metallb：

sh
helm install metallb metallb/metallb

确保 CRD 已安装：

Metallb 依赖的 IPAddressPool 和 L2Advertisement CRD 需要先被安装。你可以通过运行以下命令来安装这些 CRD：

sh
复制代码
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/main/config/crd/bases/metallb.io_ipaddresspools.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/main/config/crd/bases/metallb.io_l2advertisements.yaml
检查 CRD 安装：

确认 CRD 已经正确安装，你可以使用以下命令来检查：

sh
kubectl get crds
确认输出中包含 ipaddresspools.metallb.io 和 l2advertisements.metallb.io。

重新安装 Helm Chart：

在确保 CRD 已安装之后，可以重新运行 Helm 安装命令：

sh
helm install <release-name> <chart-name>
例如，如果你在使用 Metallb 的 Helm Chart，可以是：

sh
helm install metallb metallb/metallb

👍 其他异常解决
删除现有 CRD：

sh
kubectl delete crd ipaddresspools.metallb.io
kubectl delete crd l2advertisements.metallb.io
重新安装 Metallb：

sh
helm install metallb metallb/metallb