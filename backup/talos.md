0. 开始操作前，请认真阅读以下说明：
(1) 调整磁盘挂载配置会被Talos系统强制触发重启，请确保集群没有人在用后再执行！
(2) 由于操作磁盘为高度风险操作，数据无价，请确认精神状态良好，视力正常，能看清每一个字母后再操作！
(3) 如果在操作过程中发现返回结果异常，请立即停止操作，必要时请立即拔掉设备上的电源线，并寻求帮助！

1. 通过talosctl disks确认目标磁盘是否在位
NODE           DEV        MODEL              SERIAL   TYPE   UUID   WWID   MODALIAS      NAME   SIZE     BUS_PATH                                                                SUBSYSTEM          SYSTEM_DISK
172.16.8.192   /dev/sda   D300 240G-SSDD03   -        SSD    -      -      scsi:t-0x00   -      240 GB   /pci0000:00/0000:00:08.1/0000:12:00.2/ata2/host1/target1:0:0/1:0:0:0/   /sys/class/block   *
172.16.8.192   /dev/sdb   ST2000NM000B-2TD   -        HDD    -      -      scsi:t-0x00   -      2.0 TB   /pci0000:00/0000:00:08.1/0000:12:00.2/ata3/host2/target2:0:0/2:0:0:0/   /sys/class/block   
172.16.8.193   /dev/sda   D300 240G-SSDD03   -        SSD    -      -      scsi:t-0x00   -      240 GB   /pci0000:00/0000:00:08.1/0000:12:00.2/ata2/host1/target1:0:0/1:0:0:0/   /sys/class/block   *
172.16.8.193   /dev/sdb   ST2000NM000B-2TD   -        HDD    -      -      scsi:t-0x00   -      2.0 TB   /pci0000:00/0000:00:08.1/0000:12:00.2/ata3/host2/target2:0:0/2:0:0:0/   /sys/class/block 

（注：正常情况下，固态硬盘(SSD)应该位于/dev/sda，机械硬盘(HDD)应该位于/dev/sdb，如果/dev/sdb硬盘未在位，或者两块硬盘位置相反，请联系产品硬件组进行确认检查）

2. 通过talosctl mounts确认目标磁盘是否已被挂载
NODE           FILESYSTEM                                                   SIZE(GB)   USED(GB)   AVAILABLE(GB)   PERCENT USED   MOUNTED ON
172.16.8.192   devtmpfs                                                     16.73      0.00       16.73           0.00%          /dev
172.16.8.192   tmpfs                                                        16.77      0.01       16.77           0.03%          /run
172.16.8.192   tmpfs                                                        16.77      0.00       16.77           0.00%          /system
172.16.8.192   tmpfs                                                        0.07       0.00       0.07            0.00%          /tmp
172.16.8.192   overlay                                                      0.00       0.00       0.00            100.00%        /
172.16.8.192   rootfs                                                       16.73      0.06       16.67           0.38%          /lib/firmware
172.16.8.192   tmpfs                                                        16.77      0.00       16.77           0.00%          /dev/shm
172.16.8.192   tmpfs                                                        16.77      0.00       16.77           0.00%          /etc/cri/conf.d/hosts
172.16.8.192   overlay                                                      16.77      0.00       16.77           0.00%          /usr/etc/udev
172.16.8.192   /dev/sda5                                                    0.10       0.01       0.09            6.30%          /system/state
172.16.8.192   overlay                                                      16.77      0.00       16.77           0.00%          /usr/local/lib/containers/hello-world
172.16.8.192   /dev/sda6                                                    238.68     21.56      217.12          9.03%          /var
......

（注：快速确认可以使用 talosctl mounts | grep /dev/sdb，如果目标Node没有输出结果说明没有挂载；目标Node仅输出一行 /var/openebs/local2 说明已挂载但未投入使用，显示多行 /var/lib/kubelet/pods 说明已经有PVC在位，禁止在未处理PVC的前提下解除挂载，否则数据会丢）
172.16.8.193   /dev/sdb1                                                    1999.42    14.03      1985.39         0.70%          /var/openebs/local2
172.16.8.193   /dev/sdb1                                                    1999.42    14.03      1985.39         0.70%          /var/lib/kubelet/pods/cd77311b-b37f-4f12-9e7b-4463fcca9b3d/volumes/kubernetes.io~local-volume/pvc-7f42effa-94c8-4027-9487-f8fdd132e4af
172.16.8.193   /dev/sdb1                                                    1999.42    14.03      1985.39         0.70%          /var/lib/kubelet/pods/532725e8-267d-4edd-9200-2e3b7c7e28c5/volumes/kubernetes.io~local-volume/pvc-dd6d6410-7b6f-4ae1-b1aa-d1cd4ba81ecb
172.16.8.193   /dev/sdb1                                                    1999.42    14.03      1985.39         0.70%          /var/lib/kubelet/pods/b02f0998-929b-40cf-8489-ee2a3dc869ab/volumes/kubernetes.io~local-volume/pvc-f5e3468b-ea5a-4730-9308-249e0d96a753
172.16.8.193   /dev/sdb1                                                    1999.42    14.03      1985.39         0.70%          /var/lib/kubelet/pods/29b9ec08-56bf-404a-8d6f-5068e7103b25/volumes/kubernetes.io~local-volume/pvc-3c06cf2a-5d91-4775-a54f-06149ba515cc

3. 修改Talos MachineConfig（建议从最后一个集群开始，倒序修改，使用 talosctl edit mc -n IP 逐台修改，禁止同时修改多个节点）
在 machine: 字段下补充如下内容：

machine:
    disks:
        - device: /dev/sdb # 请多次确认，严禁写成 /dev/sda，否则节点必炸！
           partitions:
              - mountpoint: /var/openebs/local2

（注：/dev/sdb 后面不要加1，如 /dev/sdb1，Talos会自动补全）

在machine.kubelet.extraMounts 字段下补充如下内容：

machine:
  kubelet:
    extraMounts:
      - destination: /var/openebs/local2
        type: bind
        source: /var/openebs/local2
        options:
          - bind
          - rshared
          - rw

确认无误后保存退出。

注意：修改磁盘挂载会强制触发集群重启，请确保集群没有人在用后再操作！
Applied configuration with a reboot: this config change can't be applied in immediate mode
diff:   &v1alpha1.Config{
        ConfigVersion: "v1alpha1",
        ConfigDebug:   &false,
        ConfigPersist: &true,
        MachineConfig: &v1alpha1.MachineConfig{
                ... // 6 identical fields

4. 等待重启结束后，确认磁盘是否挂载在位（使用 talosctl mounts -n IP | grep /dev/sdb）确认：
172.16.8.192   /dev/sdb1                                                    1999.42    14.03      1985.39         0.70%          /var/openebs/local2
172.16.8.193   /dev/sdb1                                                    1999.42    14.03      1985.39         0.70%          /var/openebs/local2

5. 创建StorageClass，引用挂载点：
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: openebs-sc2
  annotations:
    openebs.io/cas-type: local
    storageclass.kubernetes.io/is-default-class: "false"
    cas.openebs.io/config: |
      - name: StorageType
        value: "hostpath"
      - name: BasePath
        value: "/var/openebs/local2"
provisioner: openebs.io/local
volumeBindingMode: WaitForFirstConsumer
reclaimPolicy: Delete

6. 部署业务时，使用 openebs-sc2 存储类部署业务，观察PVC是否正常挂载：
harbor            harbor-database                                         Bound    pvc-3c06cf2a-5d91-4775-a54f-06149ba515cc   1Gi        RWO            openebs-sc2         24m
harbor            harbor-jobservice                                       Bound    pvc-f5e3468b-ea5a-4730-9308-249e0d96a753   5Gi        RWO            openebs-sc2         24m
harbor            harbor-redis                                            Bound    pvc-7f42effa-94c8-4027-9487-f8fdd132e4af   1Gi        RWO            openebs-sc2         24m
harbor            harbor-registry                                         Bound    pvc-dd6d6410-7b6f-4ae1-b1aa-d1cd4ba81ecb   5Gi        RWO            openebs-sc2         24m