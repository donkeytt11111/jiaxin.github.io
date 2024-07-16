1.2.12 设置kubernetes仓库
cat > /etc/yum.repos.d/kubernetes.repo <<EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

yum makecache fast
1.3 基础部署
1.3.1 安装kubelet、kubectl、kubeadm
yum install -y kubelet-1.23.10 kubectl-1.23.10 kubeadm-1.23.10。（当前1.23最新版本为1.23.10）
systemctl enable kubelet
1.3.2 单master部署
