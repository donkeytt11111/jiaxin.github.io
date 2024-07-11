```shell
docker load -i registry.tar.gz
docker ps -a
docker run -d -p 5000:5000 --name registry regisrty:2
docker run -d -p 5000:5000 --name registry registry:2
docker load -i etcd.tar.gz
docker tag gcr.io/etcd-development/etcd:v3.5.10 172.16.8.102:5000/gcr.io/etcd-development/etcd:v3.5.10
docker push 172.16.8.102:5000/gcr.io/etcd-development/etcd:v3.5.10
vim /etc/docker/daemon.json
service docker restart
docker push 172.16.8.102:5000/gcr.io/etcd-development/etcd:v3.5.10
docker start
docker start registry.tar.gz
docker start registry
docker push 172.16.8.102:5000/gcr.io/etcd-development/etcd:v3.5.10
talosctl edit mc
talosctl image pull gcr.io/etcd-development/etcd:v3.5.10
talosctl reboot
talosctl dashboard
ping 172.16.8.30
talosctl service
talosctl kubeconfig
kubectl get nodes -o wide


docker配置

{
    "data-root": "/home/chen/docker_data",
    "insecure-registries": ["172.16.8.102:5000"]
}

http://172.16.8.102:5000/v2/gcr.io