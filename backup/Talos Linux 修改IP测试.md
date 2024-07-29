Talos Linux 修改IP测试 一、测试环境
设备型号 设备IPv4地址 设备IPv6地址 MYCNP-SV-P2C 172.16.223.101/24 2408:8631:c02:ffe3::a101/64 MYCNP-SV-P2C 172.16.223.102/24 2408:8631:c02:ffe3::a102/64 MYCNP-SV-P2C 172.16.223.103/24 2408:8631:c02:ffe3::a103/64
二、修改流程 1. 修改前准备工作
(1) 检查集群运行状态：
$ kubectl get nodes -o wide
NAME         STATUS   ROLES           AGE    VERSION   INTERNAL-IP EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION   CONTAINER-RUNTIME cncp-ms-01   Ready    control-plane   19m    v1.28.5   172.16.223.101   <none> Talos (v1.5.5-mycos2) 6.1.45-talos     containerd://1.6.23 cncp-ms-02   Ready    control-plane   11m    v1.28.5   172.16.223.102   <none> Talos (v1.5.5-mycos2) 6.1.45-talos     containerd://1.6.23 cncp-ms-03   Ready    control-plane   4m5s   v1.28.5   172.16.223.103   <none> Talos (v1.5.5-mycos2) 6.1.45-talos     containerd://1.6.23
$ talosctl etcd status
NODE             MEMBER             DB SIZE   IN USE            LEADER RAFT INDEX   RAFT TERM   RAFT APPLIED INDEX   LEARNER   ERRORS 172.16.223.101   09b46a3d5cc2c53a   6.6 MB    3.4 MB (51.96%) 09b46a3d5cc2c53a   5137 3 5137 false 172.16.223.103   dfc9e2c28fb4d3f1   6.3 MB    3.4 MB (54.55%) 09b46a3d5cc2c53a   5137 3 5137 false 172.16.223.102   da76ae5474605fb9   6.5 MB    3.4 MB (52.63%) 09b46a3d5cc2c53a   5137 3 5137 false
2. 集群停机进入维护模式
如果集群上有Pod因为PodDisruptionBudgets导致无法被Cordon（如CNCP平台中的 OpenSearch集群），可以加上 --disable-eviction 越过限制强制停机，但可能会导致数据；
正常情况下使用 --ignore-daemonsets --delete-emptydir-data 即可。
$ kubectl cordon cncp-ms-01 cncp-ms-02 cncp-ms-03
node/cncp-ms-01 cordoned node/cncp-ms-02 cordoned node/cncp-ms-03 cordoned
$ kubectl drain cncp-ms-01 cncp-ms-02 cncp-ms-03 --ignore-daemonsets --deleteemptydir-data --disable-eviction
node/cncp-ms-01 already cordoned node/cncp-ms-02 already cordoned node/cncp-ms-03 already cordoned Warning: ignoring DaemonSet-managed Pods: kube-system/cilium-6ghzx, kubesystem/cni-installation-4j59n, kube-system/kube-multus-ds-wc697, metallbsystem/speaker-8xlgg, openebs/openebs-ndm-node-exporter-m99cw, openebs/openebs-ndm-wk72w pod/cilium-operator-65968bc7c7-xnnm5 deleted pod/dashboard-metrics-scraper-f8458dcd8-5wfrk deleted pod/kubernetes-dashboard-6c94dffcdd-v68hd deleted pod/controller-6db6d84dc7-mz48d deleted pod/openebs-localpv-provisioner-b5b54dbcb-rsvbf deleted pod/openebs-ndm-cluster-exporter-5d9fb7c64f-tmj4v deleted pod/openebs-ndm-operator-648fdb55f8-8kdt7 deleted pod/reloader-reloader-f5dd7f595-bpqnc deleted pod/coredns-78f679c54d-4fmnr deleted pod/coredns-78f679c54d-ss8z6 deleted pod/nfs-server-7f4db4d9f7-99m26 deleted node/cncp-ms-01 drained Warning: ignoring DaemonSet-managed Pods: kube-system/cilium-rnfrq, kubesystem/cni-installation-h9nqg, kube-system/kube-multus-ds-fh6w7, metallbsystem/speaker-2sp5l, openebs/openebs-ndm-fjjr2, openebs/openebs-ndm-nodeexporter-gzwq4 pod/cilium-operator-65968bc7c7-7bbqf deleted node/cncp-ms-02 drained Warning: ignoring DaemonSet-managed Pods: kube-system/cilium-54v2r, kubesystem/cni-installation-g447l, kube-system/kube-multus-ds-4v9w4, metallbsystem/speaker-584kl, openebs/openebs-ndm-4q4v5, openebs/openebs-ndm-nodeexporter-lpd6j
再次确认集群K8s状态是否已进入停机维护模式：
3. 暂时移除多余的master节点上的etcd
在开始前，查看集群上etcd的状态：
重点检查两个数据：
将 除了master1以外的 （注意别删错了）所有集群执行命令，使其离开etcd集群：
推荐倒序执行（cncp-ms-03 -> cncp-ms-02）。
pod/cilium-operator-65968bc7c7-6m2mp deleted node/cncp-ms-03 drained
$ kubectl get nodes -o wide
NAME         STATUS                     ROLES           AGE   VERSION INTERNAL-IP      EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION CONTAINER-RUNTIME cncp-ms-01   Ready,SchedulingDisabled   control-plane   32m   v1.28.5   172.16.223.101   <none>        Talos (v1.5.5-mycos2) 6.1.45-talos containerd://1.6.23 cncp-ms-02   Ready,SchedulingDisabled   control-plane   24m   v1.28.5   172.16.223.102   <none>        Talos (v1.5.5-mycos2) 6.1.45-talos containerd://1.6.23 cncp-ms-03   Ready,SchedulingDisabled   control-plane   16m   v1.28.5   172.16.223.103   <none>        Talos (v1.5.5-mycos2) 6.1.45-talos containerd://1.6.23
NODE             MEMBER             DB SIZE   IN USE            LEADER RAFT INDEX   RAFT TERM   RAFT APPLIED INDEX   LEARNER   ERRORS 172.16.223.103   dfc9e2c28fb4d3f1   6.3 MB    4.0 MB (64.18%) 09b46a3d5cc2c53a   7807 3 7807 false 172.16.223.102   da76ae5474605fb9   6.5 MB    4.0 MB (61.95%) 09b46a3d5cc2c53a   7807 3 7807 false 172.16.223.101   09b46a3d5cc2c53a   6.6 MB    4.0 MB (61.29%) 09b46a3d5cc2c53a   7807 3 7807 false
RAFT INDEX/RAFT TERM/RAFT APPLIED INDEX 所有etcd成员都保持一致 所有的etcd成员都作为其中一个节点的FOLLOWER（即所有成员的LEADER一致，指向集 群内的某个MEMBER）
在 cncp-ms-03 上执行：
确认状态（由于master3已退出，所以master3连接不上属于正常现象）：
在 cncp-ms-02 上执行：
确认状态：
由于master2、master3的etcd已经停止，kubectl状态变为NotReady：
$ talosctl etcd leave -n 172.16.223.103
$ talosctl etcd status
WARNING: 1 error occurred: * 172.16.223.103: rpc error: code = DeadlineExceeded desc = failed to create etcd client: error building etcd client: context deadline exceeded
NODE             MEMBER             DB SIZE   IN USE            LEADER RAFT INDEX   RAFT TERM   RAFT APPLIED INDEX   LEARNER   ERRORS 172.16.223.101   09b46a3d5cc2c53a   6.6 MB    3.0 MB (46.24%) 09b46a3d5cc2c53a   8718 3 8718 false 172.16.223.102   da76ae5474605fb9   6.5 MB    3.0 MB (46.80%) 09b46a3d5cc2c53a   8718 3 8718 false
$ talosctl etcd leave -n 172.16.223.102
$ talosctl etcd status
WARNING: 2 errors occurred: * 172.16.223.102: rpc error: code = DeadlineExceeded desc = failed to create etcd client: error building etcd client: context deadline exceeded * 172.16.223.103: rpc error: code = DeadlineExceeded desc = failed to create etcd client: error building etcd client: context deadline exceeded
NODE             MEMBER             DB SIZE   IN USE            LEADER RAFT INDEX   RAFT TERM   RAFT APPLIED INDEX   LEARNER   ERRORS 172.16.223.101   09b46a3d5cc2c53a   6.6 MB    2.9 MB (43.93%) 09b46a3d5cc2c53a   9099 3 9099 false
4. 修改IP及必要配置
新IP配置如下：
设备型号 设备IPv4地址 设备IPv6地址 MYCNP-SV-P2C 172.16.222.101/24 2408:8631:c02:ffe2::a101/64 MYCNP-SV-P2C 172.16.222.102/24 2408:8631:c02:ffe2::a102/64 MYCNP-SV-P2C 172.16.222.103/24 2408:8631:c02:ffe2::a103/64
建议按照倒序修改(cncp-ms-03 -> cncp-ms-02 -> cncp-ms-01)
修改 cncp-ms-03 的Talos MachineConfig：
修改 cncp-ms-02 和 cncp-ms-01 的Talos MachineConfig，方法同上。
修改系统的 talosconfig ，将endpoints和nodes指向新的IP：
$ kubectl get no -o wide
NAME         STATUS                        ROLES           AGE   VERSION INTERNAL-IP      EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION CONTAINER-RUNTIME cncp-ms-01   Ready,SchedulingDisabled      control-plane   41m   v1.28.5   172.16.223.101   <none>        Talos (v1.5.5-mycos2) 6.1.45-talos containerd://1.6.23 cncp-ms-02   NotReady,SchedulingDisabled   control-plane   33m   v1.28.5   172.16.223.102   <none>        Talos (v1.5.5-mycos2) 6.1.45-talos containerd://1.6.23 cncp-ms-03   NotReady,SchedulingDisabled   control-plane   26m   v1.28.5   172.16.223.103   <none>        Talos (v1.5.5-mycos2) 6.1.45-talos containerd://1.6.23
$ talosctl edit mc -n 172.16.223.103
machine.network.interfaces: 修改address、routes、 machine.network.extraHostEntries: 修改ip为master1的地址（172.16.222.101） cluster.controlPlane.endpoint: 修改为master1的地址（https://172.16.222.101:6443） cluster.apiServer.certSANs: 修改为master1的地址 （172.16.222.101） cluster.inlineManifests: 修改为master1的地址（172.16.222.101）
修改上游交换机对应的配置，在此不做叙述。
5. 修改后恢复工作
修改完成后，通过命令，确认集群状态：
正常情况下，应该是master1 READY=true，其余节点READY=false，这是因为已经将除了 master1节点以外的所有节点etcd清除所导致的。
对 cncp-ms-02 和 cncp-ms-03 进行重启：
$ vim ~/.talos/config
context: mingyang contexts: mingyang: endpoints: - 172.16.223.101 nodes: - 172.16.223.101 - 172.16.223.102 - 172.16.223.103 ca: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tL.....
$ talosctl get machinestatus
NODE             NAMESPACE   TYPE            ID        VERSION   STAGE READY 172.16.222.101   runtime     MachineStatus   machine   28        running   true 172.16.222.102   runtime     MachineStatus   machine   25        running   false 172.16.222.103   runtime     MachineStatus   machine   28        running   false
$ talosctl reboot -n 172.16.222.102,172.16.222.103
◲ watching nodes: [172.16.222.102 172.16.222.103] * 172.16.222.102: task: stopAllPods action: START * 172.16.222.103: task: stopAllPods action: START
由于集群在此之前已经进入停机维护模式，如果为了节省时间，可以接上键盘直接按 Ctrl+Alt+Delete重启，没有键盘的话可以长按电源键关机或者直接拔电重启（不推荐）。
重启后接入Console，从Console中等待日志（注意不要直接talosctl bootstrap）：
[ 133.912456] [talos] service[etcd](Preparing): Running pre state [ 133.983361] [talos] service[kubelet](Preparing): Running pre state [ 134.071789] [talos] etcd is waiting to join the cluster, if this node is the first node in the cluster, please run `talosctl bootstrap` against one of the following IPs: [ 134.253129] [talos] [172.16.222.102 2408:8631:c02:ffe2::a102] [ 134.321948] [talos] service[kubelet](Preparing): Creating service runner [ 134.402250] [talos] service[etcd](Preparing): Creating service runner [ 138.074203] [talos] retrying error: etcdserver: rpc not supported for learner [ 142.893058] [talos] service[kubelet](Running): Health check successful [ 143.286335] [talos] service[etcd](Running): Health check failed: etcdserver: rpc not supported for learner [ 143.402049] [talos] controller failed {"component": "controller-runtime", "controller": "k8s.KubeletStaticPodController", "error": "error refreshing pod status: error fetching pod status: an error on the server (\"Authorization error (user=apiserver-kubelet-client, verb=get, resource=nodes, subresource=proxy)\") has prevented the request from succeeding"} [ 148.251526] [talos] task startAllServices (1/1): service "etcd" to be "up" [ 163.251247] [talos] task startAllServices (1/1): service "etcd" to be "up" [ 163.847161] [talos] successfully promoted etcd member [ 178.248995] [talos] task startAllServices (1/1): service "etcd" to be "up" [ 180.693804] [talos] service[etcd](Running): Health check successful [ 180.768997] [talos] task startAllServices (1/1): done, 1m2.974997497s [ 180.846205] [talos] rendered new static pod {"component": "controllerruntime", "controller": "k8s.StaticPodServerController", "id": "kubeapiserver"} [ 181.007718] [talos] phase startEverything (18/18): done, 1m3.121110227s [ 181.087097] [talos] boot sequence: done: 1m21.155004767s [ 181.150788] [talos] controller failed {"component": "controller-runtime", "controller": "k8s.ManifestApplyController", "error": "error creating mapping for object /v1/Namespace/cncp-production: Get \"https://localhost:6443/api? timeout=32s\": dial tcp [::1]:6443: connect: connection refused"} [ 181.459880] [talos] rendered new static pod {"component": "controllerruntime", "controller": "k8s.StaticPodServerController", "id": "kubecontroller-manager"} [ 181.459924] [talos] rendered new static pod {"component": "controllerruntime", "controller": "k8s.StaticPodServerController", "id": "kubescheduler"}
等待集群提示 successfully promoted etcd member
集群最终启动成功后会提示：
根据修改后的talosconfig，重新生成正确的kubeconfig：
检查K8s集群状态（由于集群还在停机维护状态，所以STATUS为 Ready,SchedulingDisabled）：
检查etcd状态：
[ 252.832068] [talos] machine is running and ready {"component": "controllerruntime", "controller": "runtime.MachineStatusController"}
$ talosctl kubeconfig -n 172.16.222.101
cluster "mingyang" already exists [(r)ename/(o)verwrite]: o auth "admin@mingyang" already exists [(r)ename/(o)verwrite]: o
$ kubectl get nodes -o wide
NAME         STATUS                     ROLES           AGE   VERSION INTERNAL-IP      EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION CONTAINER-RUNTIME cncp-ms-01   Ready,SchedulingDisabled   control-plane   82m   v1.28.5   172.16.222.101   <none>        Talos (v1.5.5-mycos2) 6.1.45-talos containerd://1.6.23 cncp-ms-02   Ready,SchedulingDisabled   control-plane   74m   v1.28.5   172.16.222.102   <none>        Talos (v1.5.5-mycos2) 6.1.45-talos containerd://1.6.23 cncp-ms-03   Ready,SchedulingDisabled   control-plane   67m   v1.28.5   172.16.222.103   <none>        Talos (v1.5.5-mycos2) 6.1.45-talos containerd://1.6.23
$ talosctl etcd status NODE             MEMBER             DB SIZE   IN USE            LEADER RAFT INDEX   RAFT TERM   RAFT APPLIED INDEX   LEARNER   ERRORS 172.16.222.101   09b46a3d5cc2c53a   6.6 MB    5.0 MB (75.23%) 09b46a3d5cc2c53a   14532 4 14532 false 172.16.222.103   b771f21d7bfa188b   6.5 MB    4.9 MB (76.19%) 09b46a3d5cc2c53a   14532 4 14532 false 172.16.222.102   e80a35afb2c9e02e   6.4 MB    4.9 MB (76.55%) 09b46a3d5cc2c53a   14532 4 14532 false
重点检查两个数据：
检查Talos就绪状态（确保所有的READY=true）：
修改K8s集群内ConfigMap，调整CIlium Endpoint：
重新启动CNI、CSI组件：
RAFT INDEX/RAFT TERM/RAFT APPLIED INDEX 所有etcd成员都保持一致 所有的etcd成员都作为其中一个节点的FOLLOWER（即所有成员的LEADER一致，指向集 群内的某个MEMBER）
$ talosctl get machinestatus 
NODE             NAMESPACE   TYPE            ID        VERSION   STAGE READY 172.16.222.101   runtime     MachineStatus   machine   28        running   true 172.16.222.102   runtime     MachineStatus   machine   27        running   true 172.16.222.103   runtime     MachineStatus   machine   20        running   true
kubectl edit configmaps -n kube-system cluster-config
apiVersion: v1 data: KUBERNETES_API_SERVER_ADDRESS: 172.16.222.101 # 修改这里 KUBERNETES_API_SERVER_PORT: "6443" kind: ConfigMap metadata: creationTimestamp: "2024-07-18T00:56:14Z" name: cluster-config namespace: kube-system resourceVersion: "324" uid: 3a7849bb-ac71-40c7-a33b-2ac809d101ba
$ kubectl rollout restart -n kube-system ds/cilium deployment/cilium-operator ds/kube-multus-ds
daemonset.apps/cilium restarted deployment.apps/cilium-operator restarted daemonset.apps/kube-multus-ds restarted
确认所有Pod都处于Running或Pending状态：
关闭维护模式，启动K8s集群：
$ kubectl rollout restart -n metallb-system ds/speaker deployment/controller
daemonset.apps/speaker restarted deployment.apps/controller restarted
$ kubectl rollout restart -n openebs ds/openebs-ndm ds/openebs-ndm-nodeexporter 
daemonset.apps/openebs-ndm restarted daemonset.apps/openebs-ndm-node-exporter restarted
$ kubectl get pods -A
NAMESPACE              NAME                                            READY STATUS    RESTARTS      AGE default                reloader-reloader-f5dd7f595-kv8m7               0/1 Pending   0             61m kube-system            cilium-bg45f                                    1/1 Running   0             2m40s kube-system            cilium-d6d9x                                    1/1 Running   0             2m40s kube-system            cilium-operator-78469dbb6d-g8fk7                1/1 Running   0             2m41s kube-system            cilium-qzmqp                                    1/1 Running   0             2m40s kube-system            cni-installation-4j59n                          1/1 Running   1 (24m ago)   88m kube-system            cni-installation-g447l                          1/1 Running   1 (18m ago)   74m kube-system            cni-installation-h9nqg                          1/1 Running   1 (18m ago)   81m kube-system            coredns-78f679c54d-mwqv2                        0/1 Pending   0             61m kube-system            coredns-78f679c54d-szrcx                        0/1 Pending   0             61m kube-system            kube-apiserver-cncp-ms-01                       1/1 Running   2 (23m ago)   23m kube-system            kube-apiserver-cncp-ms-02                       1/1 Running   0             13m
检查所有Pod的运行状态：
$ kubectl uncordon cncp-ms-01 cncp-ms-02 cncp-ms-03
node/cncp-ms-01 uncordoned node/cncp-ms-02 uncordoned node/cncp-ms-03 uncordoned
$ kubectl get pods -A
NAMESPACE              NAME                                            READY STATUS    RESTARTS      AGE default                reloader-reloader-f5dd7f595-kv8m7               1/1 Running   0             63m kube-system            cilium-bg45f                                    1/1 Running   0             4m11s kube-system            cilium-d6d9x                                    1/1 Running   0             4m11s kube-system            cilium-operator-78469dbb6d-g8fk7                1/1 Running   0             4m12s kube-system            cilium-qzmqp                                    1/1 Running   0             4m11s kube-system            cni-installation-4j59n                          1/1 Running   1 (26m ago)   90m kube-system            cni-installation-g447l                          1/1 Running   1 (19m ago)   75m kube-system            cni-installation-h9nqg                          1/1 Running   1 (19m ago)   82m kube-system            coredns-78f679c54d-mwqv2                        1/1 Running   0             63m kube-system            coredns-78f679c54d-szrcx                        1/1 Running   0             63m kube-system            kube-apiserver-cncp-ms-01                       1/1 Running   2 (25m ago)   25m kube-system            kube-apiserver-cncp-ms-02                       1/1 Running   0             14m kube-system            kube-apiserver-cncp-ms-03                       1/1 Running   0             14m kube-system            kube-controller-manager-cncp-ms-01              1/1 Running   2 (25m ago)   25m kube-system            kube-controller-manager-cncp-ms-02              1/1 Running   1 (15m ago)   14m kube-system            kube-controller-manager-cncp-ms-03              1/1 Running   1 (14m ago)   14m kube-system            kube-multus-ds-4wp6k                            1/1
Running   0             3m52s kube-system            kube-multus-ds-97lrr                            1/1 Running   0             3m52s kube-system            kube-multus-ds-lr7wx                            1/1 Running   0             4m11s kube-system            kube-scheduler-cncp-ms-01                       1/1 Running   2 (25m ago)   25m kube-system            kube-scheduler-cncp-ms-02                       1/1 Running   1 (15m ago)   14m kube-system            kube-scheduler-cncp-ms-03                       1/1 Running   1 (14m ago)   14m kube-system            manager-cncp-ms-01                              1/1 Running   0             24m kube-system            manager-cncp-ms-02                              1/1 Running   0             14m kube-system            manager-cncp-ms-03                              1/1 Running   0             14m kube-system            nfs-server-7f4db4d9f7-68tbm                     1/1 Running   0             63m kubernetes-dashboard   dashboard-metrics-scraper-f8458dcd8-5grsx       1/1 Running   0             63m kubernetes-dashboard   kubernetes-dashboard-6c94dffcdd-cpcpp           1/1 Running   0             63m metallb-system         controller-f8f87997-sc796                       1/1 Running   0             3m40s metallb-system         speaker-r77sz                                   1/1 Running   0             3m40s metallb-system         speaker-xxmxs                                   1/1 Running   0             3m40s metallb-system         speaker-ztsr6                                   1/1 Running   0             3m40s openebs                openebs-localpv-provisioner-b5b54dbcb-nhgbg     1/1 Running   0             63m openebs                openebs-ndm-947b5                               1/1 Running   0             104s openebs                openebs-ndm-9l4jx                               1/1 Running   0             107s openebs                openebs-ndm-cluster-exporter-5d9fb7c64f-d7h2x   1/1 Running   0             63m openebs                openebs-ndm-node-exporter-f7tgn                 1/1 Running   0             103s openebs                openebs-ndm-node-exporter-fqcbd                 1/1 Running   0             108s openebs                openebs-ndm-node-exporter-vwwvs                 1/1 Running   0             105s openebs                openebs-ndm-operator-8674c4679c-ptrnl           1/1
Running   0             2m1s openebs                openebs-ndm-pg2jw                               1/1 Running   0             102s