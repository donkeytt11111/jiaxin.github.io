```shell
#修改hosts
cat >>/etc/hosts<<EOF
10.42.13.201 cncp-ms-01
10.42.13.202 cncp-ms-02
10.42.13.203 cncp-ms-03
EOF

hostnamectl  set-hostname cncp-ms-01

#关闭selinux
sudo setenforce 0 #临时
sudo sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
reboot

#关闭防火墙
systemctl disable firewalld && systemctl stop firewalld

#关闭selinux
修改文件/etc/selinux/config：SELINUX=disabled。

#时间同步
yum install -y chrony
sed -i.bak '3,6d' /etc/chrony.conf && sed -i '3c server ntp1.aliyun.com iburst' /etc/chrony.conf
systemctl enable chronyd --now && systemctl restart chronyd
chronyc sources

#安装ipvs
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe ip_vs
modprobe ip_vs_rr
modprobe ip_vs_wrr
modprobe ip_vs_sh
modprobe nf_conntrack
EOF

chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules

yum -y install ipset ipvsadm

yum install -y ebtables socat ipset conntrack
```

```shell
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF


sudo yum makecache

```

#基础部署
yum install -y kubelet-1.23.10 kubectl-1.23.10 kubeadm-1.23.10
systemctl enable kubelet

```yaml
#单机 创建yaml文件
---
kind: InitConfiguration
apiVersion: kubeadm.k8s.io/v1beta3
bootstrapTokens:
- token: token0.mingyangpassw0rd
  ttl: 0s
  usages:
  - signing
  - authentication
nodeRegistration:
  name: CNCP-MS-02
  imagePullPolicy: IfNotPresent
  taints: null
localAPIEndpoint:
  advertiseAddress: 10.42.13.201
  bindPort: 6443

---
kind: ClusterConfiguration
apiVersion: kubeadm.k8s.io/v1beta3
networking:
  podSubnet: "10.32.0.0/16,fd00:1032::/48"
  serviceSubnet: "10.64.0.0/16,fd00:1064::/112"
  dnsDomain: cluster.local
kubernetesVersion: 1.23.10
controlPlaneEndpoint: "10.42.13.201:6443"
apiServer:
  certSANs:
  - 10.42.13.201
  - 10.42.13.202
  - 10.42.13.203
certificatesDir: /etc/kubernetes/pki
imageRepository: registry.aliyuncs.com/google_containers
featureGates:
  IPv6DualStack: true
clusterName: kubernetes

```
执行部署
kubeadm init --config kubeadm.yaml

kubeadm init --skip-phases=addon/kube-proxy

kubeadm init --config kubeadm.yaml --skip-phases=addon/kube-proxy

配置kubeconfig：
mkdir -p /root/.kube
cp -i /etc/kubernetes/admin.conf /root/.kube/config
chown root:root /root/.kube/config

```shell
#关联证书
cd /etc/kubernetes/pki
zip -r pki_backup.zip ca.crt ca.key sa.key sa.pub front-proxy-ca.crt front-proxy-ca.key etcd/ca.crt etcd/ca.key


/etc/kubernetes/pki/ca.crt
/etc/kubernetes/pki/ca.key
/etc/kubernetes/pki/sa.key
/etc/kubernetes/pki/sa.pub
/etc/kubernetes/pki/front-proxy-ca.crt
/etc/kubernetes/pki/front-proxy-ca.key
/etc/kubernetes/pki/etcd/ca.crt
/etc/kubernetes/pki/etcd/ca.key

分别在master02、master03节点上创建目录/etc/kubernetes/pki/etcd。
mkdir -p /etc/kubernetes/pki/etcd。
然后，将表格中所列的证书、密钥文件，从master01节点分别拷贝至master02、master03节点

```

kubeadm join 10.42.13.201:6443 --token token0.mingyangpassw0rd \
	--discovery-token-ca-cert-hash sha256:72b381dbb33e6cb50d1e204313b4c5b290c224f924e9962eaba43b82abeacbc8

👍 异常处理
rm -rf /etc/kubernetes
rm -rf /var/lib/etcd
rm -rf /var/lib/kubelet
rm -rf /var/lib/cni
rm -rf /etc/cni/net.d
iptables -F
iptables -t nat -F
iptables -t mangle -F
ipvsadm --clear
kubeadm reset