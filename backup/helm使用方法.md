```yaml
helm repo add harbor https://helm.goharbor.io
helm repo update
helm fetch harbor/harbor
tar -zxvf harbor-*.tgz
cd harbor

异常解决：
helm uninstall harbor --namespace harbor


helm install --create-namespace --namespace harbor harbor ./ \
--set expose.type=clusterIP \
--set expose.tls.auto.commonName=172.16.102.146 \
--set externalURL=https://172.16.102.146 \
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
--set 'exporter.nodeSelector.kubernetes\.io/hostname=cncp-ms-01'


