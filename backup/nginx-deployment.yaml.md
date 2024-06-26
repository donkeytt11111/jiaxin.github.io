```yaml
---
kind: PersistentVolume
apiVersion: v1
metadata:
  name: nginx
  annotations:
    helm.sh/resource-policy: keep
spec:
  capacity:
    storage: 20Gi
  nfs:
    server: endpoint
    path: /var/local-storage
  accessModes:
  - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nginxstorageclass

---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: nginx
  namespace: cncp-production
  annotations:
    helm.sh/resource-policy: keep
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 20Gi
  storageClassName: nginxstorageclass

---
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx
  namespace: cncp-production
data:
  nginx.conf: |-
    events {
        worker_connections 50000;
    }
    pid /var/run/nginx.pid;
    http {
        include       mime.types;
        default_type  application/octet-stream;
        client_max_body_size 1000m;
        sendfile        on;
        keepalive_timeout  65;
        log_format app_access_log 'logstart|$time_iso8601|$remote_addr|$remote_port|$server_addr|$server_port|$host|$http_host$uri|$status|$http_x_forwarded_for|$http_referer|$http_user_agent|$upstream_cache_status|$bytes_sent|$proxy_add_x_forwarded_for|$connection_requests|$request_length|';
    }
    stream {
    }
  init-entrypoint.sh: |-
    #!/usr/bin/env sh
    set -e -u
    NGINX_INIT_PATH="/opt/nginx"
    NGINX_CONFIG_PATH="/usr/local/nginx/conf/nginxconf"
    config_files=(nginx.conf)
    for file in ${config_files[@]}
    do
        if [ ! -f $NGINX_CONFIG_PATH/$file ]; then
            cp $NGINX_INIT_PATH/$file $NGINX_CONFIG_PATH
        fi
    done
  handle-outerchains.sh: |-
    #!/usr/bin/env sh
    set +e +u
    cd /usr/local/nginx/conf/nginxconf
    declare -A filesTime
    label=0
    while true
    do
        label=0
        echo "label"$label
        filesname=`ls -lR |grep -v ^d|awk '{print $9}' | tr -s '\n'`
        echo "allfilename"$filesname
        for filestr in ${filesname[@]};do
        echo "filename"$filestr
        if [ "$filestr" != "nginx.conf" ] && [[ $filestr != sed* ]];then
        echo "filename"$filestr
          old_modified_time=`stat -c %y $filestr`
          echo "new time"$old_modified_time
          if [ "${filesTime["$filestr"]}" != "$old_modified_time" ];then
          echo "first time"${filesTime[$filestr]}
          label=1
          echo "label"$label
          sed -i "s/'/\"/g" $filestr
          new_modified_time=`stat -c %y $filestr`
          filesTime[$filestr]=$new_modified_time
          echo "first time2"${filesTime[$filestr]}
          echo "new time2"$new_modified_time
          fi
        fi
        done
        if [ "$label" == 1 ];then
        nginx -s reload
        fi
        sleep 3
    done
  docker-entrypoint.sh: |-
    #!/usr/bin/env sh
    set -e -u
    #���cacheȨ�ޱ�������
    chmod 777 /usr/local/nginx/cache
    nohup /usr/local/bin/handle-outerchains.sh > /var/log/handle-outerchains.log 2>&1 &
    /usr/local/nginx/sbin/nginx
    /opt/reload.sh
  fluent.conf: |
    <system>
        enable_filter_chain_optimization false
    </system>
    <source>
      @id fluentd-containers.log
      @type tail
      path /usr/local/nginx/logs/*.log
      pos_file /var/log/nginx/nginx.pos
      pos_file_compaction_interval 24h
      tag nginx.*
      read_from_head true
      follow_inodes true
      <parse>
        @type none
      </parse>
    </source>
    <match nginx.**>
      @type opensearch 
      include_tag_key true 
      #host 172.16.100.198
      host opensearch-cluster-master.cncp-system
      #port 30511
      port 9200
      user admin
      password admin
      index_name cncp-log
      include_timestamp true    
      logstash_format false
      request_timeout    30s
      ssl_verify false
      timezone +08:00
      scheme http
      <buffer>
        @type file
        path /var/log/fluentd-buffers/kubernetes.system.buffer
        flush_mode interval
        retry_type exponential_backoff
        flush_thread_count 2
        flush_interval 5s
        retry_forever
        retry_max_interval 30
        chunk_limit_size 2M
        queue_limit_length 8
        overflow_action block
        timekey 1h
     </buffer>
    </match>
---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nginx
  namespace: cncp-production
spec:
  replicas: 3
  revisionHistoryLimit: 15
  selector:
    matchLabels:
      cncp-component: nginx
  template:
    metadata:
      labels:
        cncp-component: nginx
    spec:
      volumes:
      - name: localtime
        hostPath:
          path: /etc/localtime
      - name: config
        configMap:
          name: nginx
          defaultMode: 511
      - name: storage
        persistentVolumeClaim:
          claimName: nginx
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
      - name: shared-data
        emptyDir: {}
      initContainers:
      - name: init
        image: mingyangtech.com.cn/mingyang/centos:v7.9.2009
        imagePullPolicy: IfNotPresent
        volumeMounts:
        - name: config
          mountPath: /opt/nginx/nginx.conf
          subPath: nginx.conf
        - name: config
          mountPath: /usr/local/bin/entrypoint.sh
          subPath: init-entrypoint.sh
        - name: storage
          mountPath: /usr/local/nginx/conf/nginxconf/
          subPath: usr/local/nginx/conf/nginxconf/
      containers:
      - name: nginx
        image: mingyangtech.com.cn/mingyang/cncp-nginx:v1.2.1
        imagePullPolicy: IfNotPresent
        env:
        - name: TZ
          value: "Asia/Shanghai"
        ports:
        - containerPort: 80
          protocol: TCP
        volumeMounts:
        - name: localtime
          mountPath: /etc/localtime
        - name: config
          mountPath: /usr/local/bin/handle-hosts.sh
          subPath: handle-hosts.sh
        - name: config
          mountPath: /usr/local/bin/handle-outerchains.sh
          subPath: handle-outerchains.sh
        - name: config
          mountPath: /usr/local/bin/docker-entrypoint.sh
          subPath: docker-entrypoint.sh
        - name: storage
          mountPath: /usr/local/nginx/conf/nginxconf/
          subPath: usr/local/nginx/conf/nginxconf/
        - name: storage
          mountPath: /usr/local/nginx/cert
          subPath: usr/local/nginx/cert
        - name: storage
          mountPath: /usr/local/nginx/cache
          subPath: usr/local/nginx/cache
        #- name: storage
        #  mountPath: /usr/local/nginx/logs
        #  subPath: usr/local/nginx/logs
        - name: shared-data
          mountPath: /usr/local/nginx/logs
      - name: fluentd
        image: mingyangtech.com.cn/mingyang/fluentd:v1.0.0
        imagePullPolicy: IfNotPresent
        env:
        - name: TZ
          value: "Asia/Shanghai"
        env:
        - name: FLUENTD_ARGS
          value: --no-supervisor -q
        resources:
          limits:
            memory: 500Mi
          requests:
            cpu: 100m
            memory: 200Mi
        volumeMounts:
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
        - name: config
          mountPath: /fluentd/etc
        - name: shared-data
          mountPath: /usr/local/nginx/logs
        securityContext:
          privileged: true
      nodeSelector:
        kubernetes.io/os: linux
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:  # Ӳ����
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - busybox-pod
            topologyKey: kubernetes.io/hostname
      tolerations:
      - key: node.kubenetes.io/not-ready
        effect: NoExecute
        tolerationSeconds: 30

---
kind: Service
apiVersion: v1
metadata:
  name: nginx-helper
  namespace: cncp-production
spec:
  ports:
  - protocol: TCP
    port: 22
    targetPort: 22
    nodePort: 32004
  selector:
    cncp-component: nginx-helper
  type: NodePort

---
kind: Deployment
apiVersion: apps/v1
metadata:
  name: nginx-helper
  namespace: cncp-production
spec:
  replicas: 1
  selector:
    matchLabels:
      cncp-component: nginx-helper
  template: 
    metadata:
      labels: 
        cncp-component: nginx-helper
    spec:
      volumes:
      - name: localtime
        hostPath:
          path: /etc/localtime
      - name: storage
        persistentVolumeClaim:
          claimName: nginx
      containers:
      - name: helper
        image: mingyangtech.com.cn/mingyang/ssh-server:v1.0.0
        imagePullPolicy: IfNotPresent
        env:
        - name: TZ
          value: "Asia/Shanghai"
        ports:
        - containerPort: 22
        resources:
          requests:
            cpu: 10m
            memory: 40Mi
          limits:
            cpu: 500m
            memory: 2Gi
        volumeMounts:
        - name: localtime
          mountPath: /etc/localtime
        - name: storage
          mountPath: /usr/local/nginx/conf/nginxconf/
          subPath: usr/local/nginx/conf/nginxconf/
        - name: storage
          mountPath: /usr/local/nginx/cert
          subPath: usr/local/nginx/cert
        - name: storage
          mountPath: /usr/local/nginx/cache
          subPath: usr/local/nginx/cache
        - name: storage
          mountPath: /usr/local/nginx/logs
          subPath: usr/local/nginx/logs
        securityContext:
          privileged: true
      nodeSelector:
        kubernetes.io/os: linux
      tolerations:
      - operator: Exists
        effect: NoSchedule
      - operator: Exists
        effect: NoExecute
  strategy:
    type: Recreate
