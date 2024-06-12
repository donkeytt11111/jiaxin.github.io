version: v1alpha1 # Indicates the schema used to decode the contents.
debug: false # Enable verbose logging to the console.
persist: true # description: |
# Provides machine specific configuration options.
machine:
    type: controlplane # Defines the role of the machine within the cluster.
    token: jj2db4.b6zct20qwbjgcj9a # The `token` is used by a machine to join the PKI of the cluster.
    # The root certificate authority of the PKI.
    ca:
        crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJQekNCOHFBREFnRUNBaEVBdk5GTHhGdjArMWdGcVNoUnV4Mkl6REFGQmdNclpYQXdFREVPTUF3R0ExVUUKQ2hNRmRHRnNiM013SGhjTk1qUXdORE13TURFek5UTXpXaGNOTXpRd05ESTRNREV6TlRNeldqQVFNUTR3REFZRApWUVFLRXdWMFlXeHZjekFxTUFVR0F5dGxjQU1oQUdUV0d1VW5hZk9naVlyNjJvTVZNZ3p2TzFNeTQ0d3NmWGloClBjVWpPODN5bzJFd1h6QU9CZ05WSFE4QkFmOEVCQU1DQW9Rd0hRWURWUjBsQkJZd0ZBWUlLd1lCQlFVSEF3RUcKQ0NzR0FRVUZCd01DTUE4R0ExVWRFd0VCL3dRRk1BTUJBZjh3SFFZRFZSME9CQllFRk5uM3QvY2s0NUgvZHAwZQpITWdpMDVjNHRiYzZNQVVHQXl0bGNBTkJBUEJobVJYenlSSHFwQ2I4ZzFLUjIrMEVyL1h2WjNVR1JkZW9NZzk0CkMzMGVzb3RVdE9xSnFsM0FGZmxia1IxOHZqWkhOaDRvNjJKOGxuaWRQL1h1aFFNPQotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
        key: LS0tLS1CRUdJTiBFRDI1NTE5IFBSSVZBVEUgS0VZLS0tLS0KTUM0Q0FRQXdCUVlESzJWd0JDSUVJREJVYzNJcXJrdVFmTjdQTzcycytrVE5DY3hXVFBzMTV5RFpIWWFNRW5OMgotLS0tLUVORCBFRDI1NTE5IFBSSVZBVEUgS0VZLS0tLS0K
    # Extra certificate subject alternative names for the machine's certificate.
    certSANs: []
    #   # Uncomment this to enable SANs.
    #   - 10.0.0.10
    #   - 172.16.0.10
    #   - 192.168.0.10

    # Used to provide additional options to the kubelet.
    kubelet:
        image: ghcr.io/siderolabs/kubelet:v1.28.5 # The `image` field is an optional reference to an alternative kubelet image.
        # The `extraMounts` field is used to add additional mounts to the kubelet container.
        extraMounts:
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
        # The `extraConfig` field is used to provide kubelet configuration overrides.
        extraConfig:
            containerLogMaxFiles: 5
            containerLogMaxSize: 30Mi
        defaultRuntimeSeccompProfileEnabled: true # Enable container runtime default Seccomp profile.
        disableManifestsDirectory: true # The `disableManifestsDirectory` field configures the kubelet to get static pod manifests from the /etc/kubernetes/manifests directory.
        
        # # The `ClusterDNS` field is an optional reference to an alternative kubelet clusterDNS ip list.
        # clusterDNS:
        #     - 10.96.0.10
        #     - 169.254.2.53

        # # The `extraArgs` field is used to provide additional flags to the kubelet.
        # extraArgs:
        #     key: value

        # # The `nodeIP` field is used to configure `--node-ip` flag for the kubelet.
        # nodeIP:
        #     # The `validSubnets` field configures the networks to pick kubelet node IP from.
        #     validSubnets:
        #         - 10.0.0.0/8
        #         - '!10.0.0.3/32'
        #         - fdc7::/16
    # Used to provide static pod definitions to be run by the kubelet directly bypassing the kube-apiserver.
    pods:
        - apiVersion: v1
          kind: Pod
          metadata:
            name: manager
            namespace: kube-system
          spec:
            containers:
                - image: mingyangtech.com.cn/mingyang/centos7:v1.0.0
                  imagePullPolicy: IfNotPresent
                  name: manager
                  resources:
                    limits:
                        cpu: 50m
                        memory: 200Mi
                  securityContext:
                    privileged: true
                  volumeMounts:
                    - mountPath: /etc/cni
                      name: cni
            hostNetwork: true
            priorityClassName: system-node-critical
            restartPolicy: Always
            volumes:
                - hostPath:
                    path: /etc/cni
                    type: DirectoryOrCreate
                  name: cni
    # Provides machine specific network configuration options.
    network:
        hostname: cncp-ms-01 # Used to statically set the hostname for the machine.
        # `interfaces` is used to define the network interface configuration.
        interfaces:
            - interface: bond-cluster # The interface name.
              # Assigns static IP addresses to the interface.
              addresses:
                - 10.1.13.214/27
                - 2001:da8:9002:c400::1003/112
              # A list of routes associated with the interface.
              routes:
                - network: 0.0.0.0/0 # The route's network (destination).
                  gateway: 10.1.13.222 # The route's gateway (if empty, creates link scope route).
                - network: ::/0 # The route's network (destination).
                  gateway: 2001:da8:9002:c400::fffe # The route's gateway (if empty, creates link scope route).
              # Bond specific options.
              bond:
                # The interfaces that make up the bond.
                interfaces: []
                # Picks a network device using the selector.
                deviceSelectors:
                    - busPath: 0000:19:00.0 # PCI, USB bus prefix, supports matching by wildcard.
                mode: active-backup # A bond option.
              mtu: 1500 # The interface's MTU.
              
              # # Picks a network device using the selector.

              # # select a device with bus prefix 00:*.
              # deviceSelector:
              #     busPath: 00:* # PCI, USB bus prefix, supports matching by wildcard.
              # # select a device with mac address matching `*:f0:ab` and `virtio` kernel driver.
              # deviceSelector:
              #     hardwareAddr: '*:f0:ab' # Device hardware address, supports matching by wildcard.
              #     driver: virtio # Kernel driver, supports matching by wildcard.
              # # select a device with bus prefix 00:*, a device with mac address matching `*:f0:ab` and `virtio` kernel driver.
              # deviceSelector:
              #     - busPath: 00:* # PCI, USB bus prefix, supports matching by wildcard.
              #     - hardwareAddr: '*:f0:ab' # Device hardware address, supports matching by wildcard.
              #       driver: virtio # Kernel driver, supports matching by wildcard.

              # # Bridge specific options.
              # bridge:
              #     # The interfaces that make up the bridge.
              #     interfaces:
              #         - enxda4042ca9a51
              #         - enxae2a6774c259
              #     # A bridge option.
              #     stp:
              #         enabled: true # Whether Spanning Tree Protocol (STP) is enabled.

              # # Indicates if DHCP should be used to configure the interface.
              # dhcp: true

              # # DHCP specific options.
              # dhcpOptions:
              #     routeMetric: 1024 # The priority of all routes received via DHCP.

              # # Wireguard specific configuration.

              # # wireguard server example
              # wireguard:
              #     privateKey: ABCDEF... # Specifies a private key configuration (base64 encoded).
              #     listenPort: 51111 # Specifies a device's listening port.
              #     # Specifies a list of peer configurations to apply to a device.
              #     peers:
              #         - publicKey: ABCDEF... # Specifies the public key of this peer.
              #           endpoint: 192.168.1.3 # Specifies the endpoint of this peer entry.
              #           # AllowedIPs specifies a list of allowed IP addresses in CIDR notation for this peer.
              #           allowedIPs:
              #             - 192.168.1.0/24
              # # wireguard peer example
              # wireguard:
              #     privateKey: ABCDEF... # Specifies a private key configuration (base64 encoded).
              #     # Specifies a list of peer configurations to apply to a device.
              #     peers:
              #         - publicKey: ABCDEF... # Specifies the public key of this peer.
              #           endpoint: 192.168.1.2:51822 # Specifies the endpoint of this peer entry.
              #           persistentKeepaliveInterval: 10s # Specifies the persistent keepalive interval for this peer.
              #           # AllowedIPs specifies a list of allowed IP addresses in CIDR notation for this peer.
              #           allowedIPs:
              #             - 192.168.1.0/24

              # # Virtual (shared) IP address configuration.

              # # layer2 vip example
              # vip:
              #     ip: 172.16.199.55 # Specifies the IP address to be used.
            - interface: bond-oob # The interface name.
              # Bond specific options.
              bond:
                # The interfaces that make up the bond.
                interfaces: []
                # Picks a network device using the selector.
                deviceSelectors:
                    - busPath: 0000:05:00.0 # PCI, USB bus prefix, supports matching by wildcard.
                    - busPath: 0000:06:00.0 # PCI, USB bus prefix, supports matching by wildcard.
                mode: active-backup # A bond option.
              mtu: 1500 # The interface's MTU.
              dhcp: false # Indicates if DHCP should be used to configure the interface.
              
              # # Picks a network device using the selector.

              # # select a device with bus prefix 00:*.
              # deviceSelector:
              #     busPath: 00:* # PCI, USB bus prefix, supports matching by wildcard.
              # # select a device with mac address matching `*:f0:ab` and `virtio` kernel driver.
              # deviceSelector:
              #     hardwareAddr: '*:f0:ab' # Device hardware address, supports matching by wildcard.
              #     driver: virtio # Kernel driver, supports matching by wildcard.
              # # select a device with bus prefix 00:*, a device with mac address matching `*:f0:ab` and `virtio` kernel driver.
              # deviceSelector:
              #     - busPath: 00:* # PCI, USB bus prefix, supports matching by wildcard.
              #     - hardwareAddr: '*:f0:ab' # Device hardware address, supports matching by wildcard.
              #       driver: virtio # Kernel driver, supports matching by wildcard.

              # # Assigns static IP addresses to the interface.
              # addresses:
              #     - 10.5.0.0/16
              #     - 192.168.3.7

              # # A list of routes associated with the interface.
              # routes:
              #     - network: 0.0.0.0/0 # The route's network (destination).
              #       gateway: 10.5.0.1 # The route's gateway (if empty, creates link scope route).
              #     - network: 10.2.0.0/16 # The route's network (destination).
              #       gateway: 10.2.0.1 # The route's gateway (if empty, creates link scope route).

              # # Bridge specific options.
              # bridge:
              #     # The interfaces that make up the bridge.
              #     interfaces:
              #         - enxda4042ca9a51
              #         - enxae2a6774c259
              #     # A bridge option.
              #     stp:
              #         enabled: true # Whether Spanning Tree Protocol (STP) is enabled.

              # # DHCP specific options.
              # dhcpOptions:
              #     routeMetric: 1024 # The priority of all routes received via DHCP.

              # # Wireguard specific configuration.

              # # wireguard server example
              # wireguard:
              #     privateKey: ABCDEF... # Specifies a private key configuration (base64 encoded).
              #     listenPort: 51111 # Specifies a device's listening port.
              #     # Specifies a list of peer configurations to apply to a device.
              #     peers:
              #         - publicKey: ABCDEF... # Specifies the public key of this peer.
              #           endpoint: 192.168.1.3 # Specifies the endpoint of this peer entry.
              #           # AllowedIPs specifies a list of allowed IP addresses in CIDR notation for this peer.
              #           allowedIPs:
              #             - 192.168.1.0/24
              # # wireguard peer example
              # wireguard:
              #     privateKey: ABCDEF... # Specifies a private key configuration (base64 encoded).
              #     # Specifies a list of peer configurations to apply to a device.
              #     peers:
              #         - publicKey: ABCDEF... # Specifies the public key of this peer.
              #           endpoint: 192.168.1.2:51822 # Specifies the endpoint of this peer entry.
              #           persistentKeepaliveInterval: 10s # Specifies the persistent keepalive interval for this peer.
              #           # AllowedIPs specifies a list of allowed IP addresses in CIDR notation for this peer.
              #           allowedIPs:
              #             - 192.168.1.0/24

              # # Virtual (shared) IP address configuration.

              # # layer2 vip example
              # vip:
              #     ip: 172.16.199.55 # Specifies the IP address to be used.
        # Used to statically set the nameservers for the machine.
        nameservers:
            - 223.5.5.5
            - 223.6.6.6
            - 2400:3200::1
            - 2400:3200:baba::1
        # Allows for extra entries to be added to the `/etc/hosts` file
        extraHostEntries:
            - ip: 10.1.13.214 # The IP of the host.
              # The host alias.
              aliases:
                - endpoint
        
        # # Configures KubeSpan feature.
        # kubespan:
        #     enabled: true # Enable the KubeSpan feature.
    # Used to provide instructions for installations.
    install:
        disk: /dev/sda # The disk used for installations.
        image: ghcr.io/siderolabs/installer:v1.5.5-mycos1 # Allows for supplying the image used to perform the installation.
        # Allows for supplying additional system extension images to install on top of base Talos image.
        extensions:
            - image: ghcr.io/siderolabs/iscsi-tools:v0.1.1 # System extension image.
            - image: ghcr.io/nanfei-chen/hello-world-service:v1.4.0 # System extension image.
        wipe: false # Indicates if the installation disk should be wiped at installation time.
        
        # # Look up disk using disk attributes like model, size, serial and others.
        # diskSelector:
        #     size: 4GB # Disk size.
        #     model: WDC* # Disk model `/sys/block/<dev>/device/model`.
        #     busPath: /pci0000:00/0000:00:17.0/ata1/host0/target0:0:0/0:0:0:0 # Disk bus path.

        # # Allows for supplying extra kernel args via the bootloader.
        # extraKernelArgs:
        #     - talos.platform=metal
        #     - reboot=k
    # Used to configure the machine's time settings.
    time:
        disabled: false # Indicates if the time service is disabled for the machine.
        # Specifies time (NTP) servers to use for setting the system time.
        servers:
            - 10.1.13.207
    # Used to configure the machine's sysctls.
    sysctls:
        vm.max_map_count: "262144"
    # Used to configure the machine's container image registry mirrors.
    registries:
        # Specifies mirror configuration for each registry host namespace.
        mirrors:
            '*':
                # List of endpoints (URLs) for registry mirrors to use.
                endpoints:
                    - https://10.1.13.207/v2/
            docker.io:
                # List of endpoints (URLs) for registry mirrors to use.
                endpoints:
                    - https://10.1.13.207/v2/docker.io
                overridePath: true # Use the exact path specified for the endpoint (don't append /v2/).
            gcr.io:
                # List of endpoints (URLs) for registry mirrors to use.
                endpoints:
                    - https://10.1.13.207/v2/gcr.io
                overridePath: true # Use the exact path specified for the endpoint (don't append /v2/).
            ghcr.io:
                # List of endpoints (URLs) for registry mirrors to use.
                endpoints:
                    - https://10.1.13.207/v2/ghcr.io
                overridePath: true # Use the exact path specified for the endpoint (don't append /v2/).
            k8s.gcr.io:
                # List of endpoints (URLs) for registry mirrors to use.
                endpoints:
                    - https://10.1.13.207/v2/k8s.gcr.io
                overridePath: true # Use the exact path specified for the endpoint (don't append /v2/).
            mingyangtech.com.cn:
                # List of endpoints (URLs) for registry mirrors to use.
                endpoints:
                    - http://10.1.13.207/v2/mingyangtech.com.cn
                overridePath: true # Use the exact path specified for the endpoint (don't append /v2/).
            nvcr.io:
                # List of endpoints (URLs) for registry mirrors to use.
                endpoints:
                    - https://10.1.13.207/v2/nvcr.io
                overridePath: true # Use the exact path specified for the endpoint (don't append /v2/).
            quay.io:
                # List of endpoints (URLs) for registry mirrors to use.
                endpoints:
                    - https://10.1.13.207/v2/quay.io
                overridePath: true # Use the exact path specified for the endpoint (don't append /v2/).
            registry.k8s.io:
                # List of endpoints (URLs) for registry mirrors to use.
                endpoints:
                    - https://10.1.13.207/v2/registry.k8s.io
                overridePath: true # Use the exact path specified for the endpoint (don't append /v2/).
        # Specifies TLS & auth configuration for HTTPS image registries.
        config:
            10.1.13.207:
                # The TLS configuration for the registry.
                tls:
                    insecureSkipVerify: true # Skip TLS server certificate verification (not recommended).
                    
                    # # Enable mutual TLS authentication with the registry.
                    # clientIdentity:
                    #     crt: LS0tIEVYQU1QTEUgQ0VSVElGSUNBVEUgLS0t
                    #     key: LS0tIEVYQU1QTEUgS0VZIC0tLQ==
                
                # # The auth configuration for this registry.
                # auth:
                #     username: username # Optional registry authentication.
                #     password: password # Optional registry authentication.
            10.1.13.207:10080:
                # The auth configuration for this registry.
                auth:
                    username: admin # Optional registry authentication.
                    password: Passw0rd@MY # Optional registry authentication.
                
                # # The TLS configuration for the registry.
                # tls:
                #     # Enable mutual TLS authentication with the registry.
                #     clientIdentity:
                #         crt: LS0tIEVYQU1QTEUgQ0VSVElGSUNBVEUgLS0t
                #         key: LS0tIEVYQU1QTEUgS0VZIC0tLQ==
                # tls:
                #     # Enable mutual TLS authentication with the registry.
                #     clientIdentity:
                #         crt: LS0tIEVYQU1QTEUgQ0VSVElGSUNBVEUgLS0t
                #         key: LS0tIEVYQU1QTEUgS0VZIC0tLQ==
                #     insecureSkipVerify: true # Skip TLS server certificate verification (not recommended).
    # Features describe individual Talos features that can be switched on or off.
    features:
        rbac: true # Enable role-based access control (RBAC).
        stableHostname: true # Enable stable default hostname.
        apidCheckExtKeyUsage: true # Enable checks for extended key usage of client certificates in apid.
        diskQuotaSupport: true # Enable XFS project quota support for EPHEMERAL partition and user disks.
        
        # # Configure Talos API access from Kubernetes pods.
        # kubernetesTalosAPIAccess:
        #     enabled: true # Enable Talos API access from Kubernetes pods.
        #     # The list of Talos API roles which can be granted for access from Kubernetes pods.
        #     allowedRoles:
        #         - os:reader
        #     # The list of Kubernetes namespaces Talos API access is available from.
        #     allowedKubernetesNamespaces:
        #         - kube-system
    # Configures the kernel.
    kernel:
        # Kernel modules to load.
        modules:
            - name: jool # Module name.
            - name: jool_siit # Module name.
            - name: jool_common # Module name.
            - name: nfsd # Module name.
    
    # # Provides machine specific control plane configuration options.

    # # ControlPlane definition example.
    # controlPlane:
    #     # Controller manager machine specific configuration options.
    #     controllerManager:
    #         disabled: false # Disable kube-controller-manager on the node.
    #     # Scheduler machine specific configuration options.
    #     scheduler:
    #         disabled: true # Disable kube-scheduler on the node.

    # # Used to partition, format and mount additional disks.

    # # MachineDisks list example.
    # disks:
    #     - device: /dev/sdb # The name of the disk to use.
    #       # A list of partitions to create on the disk.
    #       partitions:
    #         - mountpoint: /var/mnt/extra # Where to mount the partition.
    #           
    #           # # The size of partition: either bytes or human readable representation. If `size:` is omitted, the partition is sized to occupy the full disk.

    #           # # Human readable representation.
    #           # size: 100 MB
    #           # # Precise value in bytes.
    #           # size: 1073741824

    # # Allows the addition of user specified files.

    # # MachineFiles usage example.
    # files:
    #     - content: '...' # The contents of the file.
    #       permissions: 0o666 # The file's permissions in octal.
    #       path: /tmp/file.txt # The path of the file.
    #       op: append # The operation to use

    # # The `env` field allows for the addition of environment variables.

    # # Environment variables definition examples.
    # env:
    #     GRPC_GO_LOG_SEVERITY_LEVEL: info
    #     GRPC_GO_LOG_VERBOSITY_LEVEL: "99"
    #     https_proxy: http://SERVER:PORT/
    # env:
    #     GRPC_GO_LOG_SEVERITY_LEVEL: error
    #     https_proxy: https://USERNAME:PASSWORD@SERVER:PORT/
    # env:
    #     https_proxy: http://DOMAIN\USERNAME:PASSWORD@SERVER:PORT/

    # # Used to configure the machine's sysfs.

    # # MachineSysfs usage example.
    # sysfs:
    #     devices.system.cpu.cpu0.cpufreq.scaling_governor: performance

    # # Machine system disk encryption configuration.
    # systemDiskEncryption:
    #     # Ephemeral partition encryption.
    #     ephemeral:
    #         provider: luks2 # Encryption provider to use for the encryption.
    #         # Defines the encryption keys generation and storage method.
    #         keys:
    #             - # Deterministically generated key from the node UUID and PartitionLabel.
    #               nodeID: {}
    #               slot: 0 # Key slot number for LUKS2 encryption.
    #               
    #               # # KMS managed encryption key.
    #               # kms:
    #               #     endpoint: https://192.168.88.21:4443 # KMS endpoint to Seal/Unseal the key.
    #         
    #         # # Cipher kind to use for the encryption. Depends on the encryption provider.
    #         # cipher: aes-xts-plain64

    #         # # Defines the encryption sector size.
    #         # blockSize: 4096

    #         # # Additional --perf parameters for the LUKS2 encryption.
    #         # options:
    #         #     - no_read_workqueue
    #         #     - no_write_workqueue

    # # Configures the udev system.
    # udev:
    #     # List of udev rules to apply to the udev system
    #     rules:
    #         - SUBSYSTEM=="drm", KERNEL=="renderD*", GROUP="44", MODE="0660"

    # # Configures the logging system.
    # logging:
    #     # Logging destination.
    #     destinations:
    #         - endpoint: tcp://1.2.3.4:12345 # Where to send logs. Supported protocols are "tcp" and "udp".
    #           format: json_lines # Logs format.

    # # Configures the seccomp profiles for the machine.
    # seccompProfiles:
    #     - name: audit.json # The `name` field is used to provide the file name of the seccomp profile.
    #       # The `value` field is used to provide the seccomp profile.
    #       value:
    #         defaultAction: SCMP_ACT_LOG

    # # Configures the node labels for the machine.

    # # node labels example.
    # nodeLabels:
    #     exampleLabel: exampleLabelValue
# Provides cluster specific configuration options.
cluster:
    id: I9fUXHhy5mqFVSAQV_lsuwwUxx3O-drZcStM16d62-E= # Globally unique identifier for this cluster (base64 encoded random 32 bytes).
    secret: 8YE+YZiG8lnSRDkYcouBhMI1Dl+safRsfarKxydViBo= # Shared secret of cluster (base64 encoded random 32 bytes).
    # Provides control plane specific configuration options.
    controlPlane:
        endpoint: https://10.1.13.214:6443 # Endpoint is the canonical controlplane endpoint, which can be an IP address or a DNS hostname.
    clusterName: mingyang # Configures the cluster's name.
    # Provides cluster specific network configuration options.
    network:
        # The CNI used.
        cni:
            name: custom # Name of CNI to use.
            # URLs containing manifests to apply for the CNI.
            urls:
                - http://10.1.13.207:8088/mingyang/cncp/cilium-dsr.yaml
        dnsDomain: cluster.local # The domain used by Kubernetes DNS.
        # The pod subnet CIDR.
        podSubnets:
            - 10.32.0.0/16
            - fd00:1032::/48
        # The service subnet CIDR.
        serviceSubnets:
            - 10.64.0.0/16
            - fd00:1064::/112
    token: 1f1vya.rhaai2peuebr05su # The [bootstrap token](https://kubernetes.io/docs/reference/access-authn-authz/bootstrap-tokens/) used to join the cluster.
    secretboxEncryptionSecret: zp3K6SyWptnHMI6yw9RX5EMJIe8A8/5jcFTdauUY6WQ= # A key used for the [encryption of secret data at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/).
    # The base64 encoded root certificate authority used by Kubernetes.
    ca:
        crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJpekNDQVRDZ0F3SUJBZ0lSQU1TdnNFNmtXZmR4RUJDSjE2dWNNNVF3Q2dZSUtvWkl6ajBFQXdJd0ZURVQKTUJFR0ExVUVDaE1LYTNWaVpYSnVaWFJsY3pBZUZ3MHlOREEwTXpBd01UTTFNek5hRncwek5EQTBNamd3TVRNMQpNek5hTUJVeEV6QVJCZ05WQkFvVENtdDFZbVZ5Ym1WMFpYTXdXVEFUQmdjcWhrak9QUUlCQmdncWhrak9QUU1CCkJ3TkNBQVRibjRaOWdOVEorMVhPNStjdWNHWDJCMks3RitCbzhub0crWHNWeHRicjhodmhudytzalNQakN6cXAKN29tYnBUUzArR044TjZrcVRrUitZQVFQUVhqcm8yRXdYekFPQmdOVkhROEJBZjhFQkFNQ0FvUXdIUVlEVlIwbApCQll3RkFZSUt3WUJCUVVIQXdFR0NDc0dBUVVGQndNQ01BOEdBMVVkRXdFQi93UUZNQU1CQWY4d0hRWURWUjBPCkJCWUVGR2wwSE1CRE5nNk53NTZGeFFJNHhGREtvYVdZTUFvR0NDcUdTTTQ5QkFNQ0Ewa0FNRVlDSVFEVWI0OHAKdHRjNGdQZk9mV2ZlWitoRUY3VnJuNTZSQ3lEQlBvSW5ENnhjZkFJaEFQSklyT2FSdUV1Z25GbU1GSnlDdVE0LwpuWnBzYXlxa1JQOU9WR1RRY0IyZgotLS0tLUVORCBDRVJUSUZJQ0FURS0tLS0tCg==
        key: LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1IY0NBUUVFSUw0eUd4QnZ3eFp5M1g3QmdXSXBtQ0VoV1lqTkRsdW9yRFBPc25zaEpmRUFvQW9HQ0NxR1NNNDkKQXdFSG9VUURRZ0FFMjUrR2ZZRFV5ZnRWenVmbkxuQmw5Z2RpdXhmZ2FQSjZCdmw3RmNiVzYvSWI0WjhQckkwago0d3M2cWU2Sm02VTB0UGhqZkRlcEtrNUVmbUFFRDBGNDZ3PT0KLS0tLS1FTkQgRUMgUFJJVkFURSBLRVktLS0tLQo=
    # The base64 encoded aggregator certificate authority used by Kubernetes for front-proxy certificate generation.
    aggregatorCA:
        crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJYakNDQVFXZ0F3SUJBZ0lRYkVYcmQvOUxhMmM4a2xWMzBqR0xBekFLQmdncWhrak9QUVFEQWpBQU1CNFgKRFRJME1EUXpNREF4TXpVek0xb1hEVE0wTURReU9EQXhNelV6TTFvd0FEQlpNQk1HQnlxR1NNNDlBZ0VHQ0NxRwpTTTQ5QXdFSEEwSUFCTDFJbGVlaTBYOTlwY0poOHcwT2tSWHRlK1luMVhzSkc5bk51MC8xcGkwaHEzUVdJMUdpCkVsL0RsZHdGQ2hjV21DcXRDUmY2OC9zdlZkNlNId1JtWEkyallUQmZNQTRHQTFVZER3RUIvd1FFQXdJQ2hEQWQKQmdOVkhTVUVGakFVQmdnckJnRUZCUWNEQVFZSUt3WUJCUVVIQXdJd0R3WURWUjBUQVFIL0JBVXdBd0VCL3pBZApCZ05WSFE0RUZnUVU3cmVBRytpdGRXQTdYeXkrS25rb1g2K0VVV1l3Q2dZSUtvWkl6ajBFQXdJRFJ3QXdSQUlnCmZnRHAzQ0Qxa094SWpmNVRpUHZlOVlZY2tqWENmQmZ5dlpKYmFBdExiVFlDSUdoZlNoVGtjaWVuNUhWUE1PVzIKN3FoeDJXVjFLdXhheEVtQ0Y0dXd5d3BLCi0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
        key: LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1IY0NBUUVFSU5aeG52MHNwMEdZaVQ1Z2ozb0JhSWQxTW00ZXlraVpnR1NqaUQ3dCszWGRvQW9HQ0NxR1NNNDkKQXdFSG9VUURRZ0FFdlVpVjU2TFJmMzJsd21IekRRNlJGZTE3NWlmVmV3a2IyYzI3VC9XbUxTR3JkQllqVWFJUwpYOE9WM0FVS0Z4YVlLcTBKRi9yeit5OVYzcElmQkdaY2pRPT0KLS0tLS1FTkQgRUMgUFJJVkFURSBLRVktLS0tLQo=
    # The base64 encoded private key for service account token generation.
    serviceAccount:
        key: LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1IY0NBUUVFSU5kYmpCR1h2dFkyVCt3RVF5bW83UTNqSElBcTZRaTEvQlNLWU44eExTOVFvQW9HQ0NxR1NNNDkKQXdFSG9VUURRZ0FFT0JVOFVmUmVNbWVobkY2U0FaS3JLMXhUNEZWT0FDVnpabFI2TnVDZlp1RzNuK21Hdko4ZgpVbFJTaUw5aDl4L1NIZ2JvN20xMWtLOGFFOE5kczBaRjN3PT0KLS0tLS1FTkQgRUMgUFJJVkFURSBLRVktLS0tLQo=
    # API server specific configuration options.
    apiServer:
        image: registry.k8s.io/kube-apiserver:v1.28.5 # The container image used in the API server manifest.
        # Extra certificate subject alternative names for the API server's certificate.
        certSANs:
            - 10.1.13.214
        disablePodSecurityPolicy: true # Disable PodSecurityPolicy in the API server and default manifests.
        # Configure the API server audit policy.
        auditPolicy:
            apiVersion: audit.k8s.io/v1
            kind: Policy
            rules:
                - level: Metadata
        
        # # Configure the API server admission plugins.
        # admissionControl:
        #     - name: PodSecurity # Name is the name of the admission controller.
        #       # Configuration is an embedded configuration object to be used as the plugin's
        #       configuration:
        #         apiVersion: pod-security.admission.config.k8s.io/v1alpha1
        #         defaults:
        #             audit: restricted
        #             audit-version: latest
        #             enforce: baseline
        #             enforce-version: latest
        #             warn: restricted
        #             warn-version: latest
        #         exemptions:
        #             namespaces:
        #                 - kube-system
        #             runtimeClasses: []
        #             usernames: []
        #         kind: PodSecurityConfiguration
    # Controller manager server specific configuration options.
    controllerManager:
        image: registry.k8s.io/kube-controller-manager:v1.28.5 # The container image used in the controller manager manifest.
    # Kube-proxy server-specific configuration options
    proxy:
        disabled: true # Disable kube-proxy deployment on cluster bootstrap.
        
        # # The container image used in the kube-proxy manifest.
        # image: registry.k8s.io/kube-proxy:v1.28.3
    # Scheduler server specific configuration options.
    scheduler:
        image: registry.k8s.io/kube-scheduler:v1.28.5 # The container image used in the scheduler manifest.
    # Configures cluster member discovery.
    discovery:
        enabled: true # Enable the cluster membership discovery feature.
        # Configure registries used for cluster member discovery.
        registries:
            # Kubernetes registry uses Kubernetes API server to discover cluster members and stores additional information
            kubernetes:
                disabled: true # Disable Kubernetes discovery registry.
            # Service registry is using an external service to push and pull information about cluster members.
            service: {}
            # # External service endpoint.
            # endpoint: https://discovery.talos.dev/
    # Etcd specific configuration options.
    etcd:
        # The `ca` is the root certificate authority of the PKI.
        ca:
            crt: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUJmakNDQVNTZ0F3SUJBZ0lSQU9XQ0JTWXNibU5pVEJQVnllajNXQmd3Q2dZSUtvWkl6ajBFQXdJd0R6RU4KTUFzR0ExVUVDaE1FWlhSalpEQWVGdzB5TkRBME16QXdNVE0xTXpOYUZ3MHpOREEwTWpnd01UTTFNek5hTUE4eApEVEFMQmdOVkJBb1RCR1YwWTJRd1dUQVRCZ2NxaGtqT1BRSUJCZ2dxaGtqT1BRTUJCd05DQUFUY0NXUEhheUM1CnE2T2FBcDdUSlRQZEgrM2FPeTFSdDN5NVFvNHJwMFprdlY5UU9uVUw3S3ZoSEVIa2ZBeGE4Tyt0eGZBNUtCMVoKQlhxeWhERWhQWmpKbzJFd1h6QU9CZ05WSFE4QkFmOEVCQU1DQW9Rd0hRWURWUjBsQkJZd0ZBWUlLd1lCQlFVSApBd0VHQ0NzR0FRVUZCd01DTUE4R0ExVWRFd0VCL3dRRk1BTUJBZjh3SFFZRFZSME9CQllFRkFMbUo3NGNsVE5BCi9SMHlRWno1UVhIUDFkVmVNQW9HQ0NxR1NNNDlCQU1DQTBnQU1FVUNJUURqNmk0SklPdE1XMzRWWisxTndyWG8KcDZPTE0yM0o0b1g5T1ZFekU3eGltZ0lnVUozaDk3MUNKV0FmUmJFSG1qd1BGcXJvMXhUZTRxekJtZXYvTytFRgp1NVk9Ci0tLS0tRU5EIENFUlRJRklDQVRFLS0tLS0K
            key: LS0tLS1CRUdJTiBFQyBQUklWQVRFIEtFWS0tLS0tCk1IY0NBUUVFSUdpOXZkK24rN3NWM2wzZTBZcGR6UEVhNWtiaVlsY0JVZENCbmFWYTZFTUZvQW9HQ0NxR1NNNDkKQXdFSG9VUURRZ0FFM0Fsangyc2d1YXVqbWdLZTB5VXozUi90MmpzdFViZDh1VUtPSzZkR1pMMWZVRHAxQyt5cgo0UnhCNUh3TVd2RHZyY1h3T1NnZFdRVjZzb1F4SVQyWXlRPT0KLS0tLS1FTkQgRUMgUFJJVkFURSBLRVktLS0tLQo=
        
        # # The container image used to create the etcd service.
        # image: gcr.io/etcd-development/etcd:v3.5.10

        # # The `advertisedSubnets` field configures the networks to pick etcd advertised IP from.
        # advertisedSubnets:
        #     - 10.0.0.0/8
    # A list of urls that point to additional manifests.
    extraManifests:
        - http://10.1.13.207:8088/mingyang/cncp/cni-installation-ds.yaml
        - http://10.1.13.207:8088/mingyang/cncp/multus-ds.yaml
        - http://10.1.13.207:8088/mingyang/cncp/openebs.yaml
        - http://10.1.13.207:8088/mingyang/cncp/openebs-sc.yaml
        - http://10.1.13.207:8088/mingyang/cncp/dashboard-deploy.yaml
        - http://10.1.13.207:8088/mingyang/cncp/metallb-native.yaml
        - http://10.1.13.207:8088/mingyang/cncp/reloader.yaml
        - http://10.1.13.207:8088/mingyang/cncp/cncp-namespaces.yaml
        - http://10.1.13.207:8088/mingyang/cncp/nfs-server-deploy.yaml
    # A list of inline Kubernetes manifests.
    inlineManifests:
        - name: cluster-config-cm # Name of the manifest.
          contents: |- # Manifest contents as a string.
            kind: ConfigMap
            apiVersion: v1
            metadata:
              name: cluster-config
              namespace: kube-system
            data:
              KUBERNETES_API_SERVER_ADDRESS: "10.1.13.214"
              KUBERNETES_API_SERVER_PORT: "6443"
    # Settings for admin kubeconfig generation.
    adminKubeconfig:
        certLifetime: 87600h0m0s # Admin kubeconfig certificate lifetime (default is 1 year).
    allowSchedulingOnControlPlanes: true # Allows running workload on control-plane nodes.
    
    # # A key used for the [encryption of secret data at rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/).

    # # Decryption secret example (do not use in production!).
    # aescbcEncryptionSecret: z01mye6j16bspJYtTB/5SFX8j7Ph4JXxM2Xuu4vsBPM=

    # # Core DNS specific configuration options.
    # coreDNS:
    #     image: registry.k8s.io/coredns/coredns:v1.10.1 # The `image` field is an override to the default coredns image.

    # # External cloud provider configuration.
    # externalCloudProvider:
    #     enabled: true # Enable external cloud provider.
    #     # A list of urls that point to additional manifests for an external cloud provider.
    #     manifests:
    #         - https://raw.githubusercontent.com/kubernetes/cloud-provider-aws/v1.20.0-alpha.0/manifests/rbac.yaml
    #         - https://raw.githubusercontent.com/kubernetes/cloud-provider-aws/v1.20.0-alpha.0/manifests/aws-cloud-controller-manager-daemonset.yaml

    # # A map of key value pairs that will be added while fetching the extraManifests.
    # extraManifestHeaders:
    #     Token: "1234567"
    #     X-ExtraInfo: info
