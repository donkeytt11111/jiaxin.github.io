```yaml
- op: add
  path: /machine/install/extensions
  value:
    - image: ghcr.io/siderolabs/iscsi-tools:v0.1.1
    - image: ghcr.io/nanfei-chen/hello-world-service:v1.4.0
- op: replace
  path: /machine/install/image
  value: ghcr.io/siderolabs/installer:v1.5.5-mycos1
- op: add
  path: /machine/kubelet/extraMounts
  value:
    - destination: /var/openebs/local
      type: bind
      source: /var/openebs/local
      options:
        - bind
        - rshared
        - rw
    - destination: /var/local-storage
      type: bind
      source: /var/local-storage
      options:
        - bind
        - rshared
        - rw
- op: add
  path: /machine/kubelet/extraConfig
  value:
    containerLogMaxFiles: 5
    containerLogMaxSize: 30Mi
- op: add
  path: /machine/network/hostname
  value: cncp-ms-01
- op: add
  path: /machine/network/interfaces
  value: 
    - interface: bond-cluster
      addresses:
      - 172.16.102.102/24
      - 2408:8631:c02:ffa2::102/64
      routes:
      - network: 0.0.0.0/0
        gateway: 172.16.102.1
      - network: ::/0
        gateway: 2408:8631:c02:ffa2::1
      mtu: 1500
      bond:
        deviceSelectors:
        - busPath: 0000:07:00.0
        - busPath: 0000:08:00.0
        mode: active-backup
    - interface: bond-ip6
      bond:
        deviceSelectors:
        - busPath: 0000:0b:00.0
        - busPath: 0000:0c:00.0
        mode: active-backup
    - interface: bond-ip4
      bond:
        deviceSelectors:
        - busPath: 0000:12:00.0
        - busPath: 0000:13:00.0
        mode: active-backup
- op: replace
  path: /machine/registries
  value:
    mirrors:
      "*":
        endpoints:
          - https://172.16.1.200/v2/
      docker.io:
        endpoints:
          - https://172.16.1.200/v2/docker.io
        overridePath: true
      gcr.io:
        endpoints:
          - https://172.16.1.200/v2/gcr.io
        overridePath: true
      ghcr.io:
        endpoints:
          - https://172.16.1.200/v2/ghcr.io
        overridePath: true
      k8s.gcr.io:
        endpoints:
          - https://172.16.1.200/v2/k8s.gcr.io
        overridePath: true
      nvcr.io:
        endpoints:
          - https://172.16.1.200/v2/nvcr.io
        overridePath: true
      quay.io:
        endpoints:
          - https://172.16.1.200/v2/quay.io
        overridePath: true
      registry.k8s.io:
        endpoints:
          - https://172.16.1.200/v2/registry.k8s.io
        overridePath: true
      mingyangtech.com.cn:
        endpoints:
          - https://172.16.1.200/v2/mingyangtech.com.cn
        overridePath: true
    config:
      172.16.1.200:
        tls:
          insecureSkipVerify: true
      172.16.106.11:10080:
        auth:
          username: admin
          password: Passw0rd@MY
- op: add
  path: /machine/kernel
  value: 
    modules:
      - name: nfsd
      - name: ngbe
      - name: txgbe
      - name: jool
      - name: jool_siit
      - name: jool_common
- op: add
  path: /machine/network/nameservers
  value: 
    - 114.114.114.114
    - 8.8.8.8
    - 2400:3200::1
    - 2400:3200:baba::1
- op: add
  path: /machine/network/extraHostEntries
  value: 
    - ip: 172.16.102.102
      aliases:
        - endpoint
        - cncp-ms-01
- op: add
  path: /machine/pods
  value:
  - kind: Pod
    apiVersion: v1
    metadata:
      name: manager
      namespace: kube-system
    spec:
      volumes:
      - name: cni
        hostPath:
          path: /etc/cni
          type: DirectoryOrCreate  
      containers:
      - name: manager
        image: mingyangtech.com.cn/mingyang/centos7:v1.0.0
        imagePullPolicy: IfNotPresent
        resources:
          limits:
            cpu: 50m
            memory: 200Mi
        volumeMounts:
        - name: cni
          mountPath: /etc/cni
        securityContext:
          privileged: true
      restartPolicy: Always
      hostNetwork: true
      priorityClassName: system-node-critical
- op: add
  path: /machine/time
  value:
    disabled: false
    servers:
    - cn.ntp.org.cn
- op: add
  path: /machine/nodeLabels   
  value:
    node-role.kubernetes.io/master: "" 
- op: add
  path: /machine/sysctls 
  value:
    vm.max_map_count: "262144"
- op: replace
  path: /cluster/network/podSubnets
  value:
    - 10.32.0.0/16
    - fd00:1032::/48
- op: replace
  path: /cluster/network/serviceSubnets
  value:
    - 10.64.0.0/16
    - fd00:1064::/112
- op: add
  path: /cluster/network/cni
  value:
    name: custom
    urls:
      - http://172.16.106.11:8088/mingyang/cncp/cilium-dsr.yaml
- op: replace
  path: /cluster/apiServer/admissionControl
  value:
   []
- op: replace
  path: /cluster/proxy
  value:
    disabled: true
    image: registry.k8s.io/kube-proxy:v1.27.1
    mode: ipvs
    extraArgs:
      ipvs-strict-arp: true
- op: add
  path: /cluster/extraManifests
  value:
  - http://172.16.106.11:8088/mingyang/cncp/cni-installation-ds.yaml
  - http://172.16.106.11:8088/mingyang/cncp/multus-with-patch.yaml
  - http://172.16.106.11:8088/mingyang/cncp/openebs.yaml
  - http://172.16.106.11:8088/mingyang/cncp/openebs-sc.yaml
  - http://172.16.106.11:8088/mingyang/cncp/dashboard-deploy.yaml
  - http://172.16.106.11:8088/mingyang/cncp/metallb-native.yaml
  - http://172.16.106.11:8088/mingyang/cncp/reloader.yaml
  - http://172.16.106.11:8088/mingyang/cncp/cncp-namespaces.yaml
  - http://172.16.106.11:8088/mingyang/cncp/nfs-server-deploy.yaml
- op: add
  path: /cluster/inlineManifests
  value:
  - name: cluster-config-cm
    contents: |-
      kind: ConfigMap
      apiVersion: v1
      metadata:
        name: cluster-config
        namespace: kube-system
      data:
        KUBERNETES_API_SERVER_ADDRESS: "172.16.102.102"
        KUBERNETES_API_SERVER_PORT: "6443"
- op: add
  path: /cluster/adminKubeconfig
  value:
    certLifetime: 87600h0m0s
- op: add
  path: /cluster/allowSchedulingOnControlPlanes
  value: true
  