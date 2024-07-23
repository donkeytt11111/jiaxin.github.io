```shell
#harbor自动化heml部署
#!/bin/bash

# 定义错误处理函数
handle_error() {
  echo "发生错误: $1"
  exit 1
}

# 检查外部URL参数
if [ -z "$1" ]; then
  echo "请提供外部URL作为参数，例如:sh test.sh https://172.16.102.146 请提供 loadBalancerIP 作为第二个参数，例如: 172.16.102.79" 
  kubectl get IPAddressPool -A
  exit 1
fi

EXTERNAL_URL="$1"

# 检查 loadBalancerIP 参数
if [ -z "$2" ]; then
  echo "请提供 loadBalancerIP 作为第二个参数，例如: 172.16.102.79"
  exit 1
fi

LOAD_BALANCER_IP="$2"

# 检查是否已有Harbor安装
echo "检查是否有已存在的Harbor安装..."
if helm ls --namespace harbor | grep -q harbor; then
  echo "正在卸载现有的Harbor安装..."
  if ! helm uninstall harbor --namespace harbor; then
    handle_error "卸载现有Harbor安装失败"
  fi
fi

# 安装Harbor
echo "正在安装Harbor..."
if ! helm install \
    --create-namespace \
    --namespace harbor \
    harbor ./ \
    --set expose.type=clusterIP \
    --set expose.tls.auto.commonName=$(echo $EXTERNAL_URL | awk -F'://' '{print $NF}') \
    --set externalURL=$EXTERNAL_URL \
    --set trivy.enabled=false \
    --set metrics.enabled=true \
    --set persistence.persistentVolumeClaim.registry.storageClass=openebs-sc \
    --set persistence.persistentVolumeClaim.jobservice.jobLog.storageClass=openebs-sc \
    --set persistence.persistentVolumeClaim.database.storageClass=openebs-sc \
    --set persistence.persistentVolumeClaim.redis.storageClass=openebs-sc \
    --set persistence.persistentVolumeClaim.trivy.storageClass=openebs-sc \
    --set 'nginx.nodeSelector.kubernetes\.io/hostname=cncp-ms-01' \
    --set 'portal.nodeSelector.kubernetes\.io/hostname=cncp-ms-01' \
    --set 'core.nodeSelector.kubernetes\.io/hostname=cncp-ms-01' \
    --set 'jobservice.nodeSelector.kubernetes\.io/hostname=cncp-ms-01' \
    --set 'registry.nodeSelector.kubernetes\.io/hostname=cncp-ms-01' \
    --set 'trivy.nodeSelector.kubernetes\.io/hostname=cncp-ms-01' \
    --set 'database.nodeSelector.kubernetes\.io/hostname=cncp-ms-01' \
    --set 'redis.nodeSelector.kubernetes\.io/hostname=cncp-ms-01' \
    --set 'exporter.nodeSelector.kubernetes\.io/hostname=cncp-ms-01' \
    --set harbor-core.image.repository=myharborregistry/harbor-core \
    --set harbor-core.image.tag=v2.3.0; then
  handle_error "安装Harbor失败"
fi

echo "Harbor安装完成!"

cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: harbor-loadbalancer
  namespace: harbor
spec:
  selector:
    app: harbor
    component: nginx
  ipFamilies:
  - IPv4
  ipFamilyPolicy: SingleStack
  ports:
  - name: http
    port: 80
    targetPort: 8080
  - name: https
    port: 443
    targetPort: 8443
  type: LoadBalancer
  loadBalancerIP: ${LOAD_BALANCER_IP}
  allocateLoadBalancerNodePorts: false
EOF
```