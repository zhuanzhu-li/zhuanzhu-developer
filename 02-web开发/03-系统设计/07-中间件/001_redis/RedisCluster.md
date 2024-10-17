[toc]

参考：https://redis.io/docs/management/scaling/

# Redis Cluster

## 一、简述

集群数据分片： hash槽 16384

支持多键操作，但需要涉及到的所有键在同一个 `hash槽` ，用户可以指定 `hash slot` 将多个键放在同一个 `hash槽` 中
例如，键`user:{123}:profile`和`user:{123}:account`保证在同一个散列槽中，因为它们共享相同的散列标签。因此，您可以在同一个多键操作中对这两个键进行操作。

* 不保证强一致性

  异步复制

  可以在必要时使用同步写入， 使用 **WAIT** 命令实现，会降低写入性能，但也不能保证强一致性

* 写入丢失场景

  网络分区期间，其中客户端与少数实例隔离，至少包括一个主实例。

  以我们的 6 节点集群为例，由 A、B、C、A1、B1、C1 组成，有 3 个主节点和 3 个副本节点。还有一个客户端，我们称之为 Z1。

  发生分区后，有可能分区的一侧有A、C、A1、B1、C1，另一侧有B、Z1。

  Z1 仍然能够写入 B，B 将接受其写入。如果分区在很短的时间内恢复正常，集群将继续正常运行。但是，如果分区持续足够长的时间让 B1 在分区的多数端被提升为主节点，则 Z1 在此期间发送给 B 的写入将丢失。

  ~~~
  Z1 可以发送到 B 的写入量有一个最大窗口：如果经过足够的时间分区的多数方选择一个副本作为主节点，则少数方的每个主节点都将停止接受写入.
  这个时间量是Redis Cluster的一个非常重要的配置指令，称为节点超时时间。
  节点超时结束后，主节点被认为发生故障，可以由其副本之一替换。类似地，在节点超时过去但主节点无法感知大多数其他主节点的情况下，它会进入错误状态并停止接受写入。
  ~~~

## 二、集群参数配置

- **cluster-enabled`<yes/no>`**：如果是，则在特定的 Redis 实例中启用 Redis 集群支持。否则实例将像往常一样作为独立实例启动。
- **cluster-config-file`<filename>`**：请注意，尽管有此选项的名称，但这不是用户可编辑的配置文件，而是 Redis 集群节点在每次发生更改时自动保存集群配置（基本上是状态）的文件，为了能够在启动时重新读取它。该文件列出了集群中的其他节点、它们的状态、持久变量等内容。由于接收到某些消息，此文件通常会被重写并刷新到磁盘上。
- **cluster-node-timeout`<milliseconds>`**：Redis 集群节点可以不可用的最长时间，不会被视为失败。如果主节点在超过指定时间段内无法访问，它将由其副本进行故障转移。此参数控制 Redis 集群中的其他重要内容。值得注意的是，在指定时间内无法到达大多数主节点的每个节点都将停止接受查询。
- **cluster-slave-validity-factor`<factor>`**：如果设置为零，副本将始终认为自己有效，因此将始终尝试对主服务器进行故障转移，而不管主服务器和副本之间的链接保持断开状态的时间长短。如果该值为正数，则计算最大断开连接时间作为*节点超时*值乘以此选项提供的因子，如果节点是副本，则如果主链接断开连接的时间超过指定的时间，它将不会尝试启动故障转移。例如，如果节点超时设置为 5 秒且有效性因子设置为 10，则与主服务器断开连接超过 50 秒的副本将不会尝试对其主服务器进行故障转移。请注意，如果没有能够对其进行故障转移的副本，则任何非零值都可能导致 Redis 集群在主服务器发生故障后不可用。在这种情况下，只有当原来的主节点重新加入集群时，集群才会恢复可用。
- **cluster-migration-barrier`<count>`**：一个主节点将保持连接的最小副本数，以便另一个副本迁移到不再被任何副本覆盖的主节点。有关详细信息，请参阅本教程中有关副本迁移的相应部分。
- **cluster-require-full-coverage`<yes/no>`**：如果将其设置为 yes，默认情况下，如果任何节点未覆盖一定百分比的键空间，则集群将停止接受写入。如果该选项设置为 no，即使只能处理有关键子集的请求，集群仍会为查询提供服务。
- **cluster-allow-reads-when-down`<yes/no>`**：如果设置为 no，默认情况下，Redis 集群中的节点将在集群标记为失败时停止为所有流量提供服务，或者当节点无法访问时master 的法定人数或未满足全覆盖时。这可以防止从不知道集群更改的节点读取可能不一致的数据。此选项可以设置为 yes 以允许在故障状态期间从节点读取，这对于想要优先读取可用性但仍希望防止不一致写入的应用程序很有用。它也可以用于只有一个或两个分片的 Redis 集群，因为它允许节点在主节点发生故障但无法进行自动故障转移时继续提供写入服务。

## 三、集群k8s搭建

~~~yaml
kind: Secret
apiVersion: v1
metadata:
  name: redis
  namespace: default
data:
  # dgisserver@redis4522
  REDIS_PASS: ZGdpc3NlcnZlckByZWRpczQ1MjI=
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: redis-cluster
  namespace: default
data:
  update-node.sh: |
    #!/bin/sh
    REDIS_NODES="/data/nodes.conf"
    sed -i -e "/myself/ s/:0@0/${POD_IP}:6379@16379/" ${REDIS_NODES}
    sed -i -e "/myself/ s/[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}/${POD_IP}/" ${REDIS_NODES}
    exec "$@"
  redis.conf: |+
    cluster-enabled yes
    cluster-require-full-coverage no
    cluster-node-timeout 15000
    cluster-config-file /data/nodes.conf
    cluster-migration-barrier 1
    appendonly no
    save ""
    protected-mode no
    maxmemory 32GB
    maxmemory-policy volatile-lru
    tcp-keepalive 60
    timeout 0
    masterauth dgisserver@redis4522
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis-cluster
  namespace: default
spec:
  serviceName: redis-cluster
  replicas: 6
  selector:
    matchLabels:
      app: redis-cluster
  template:
    metadata:
      labels:
        app: redis-cluster
    spec:
      # affinity:
      #   nodeAffinity: 
      #     requiredDuringSchedulingIgnoredDuringExecution:
      #       nodeSelectorTerms:
      #       - matchExpressions:
      #         - key: "kubernetes.io/hostname"
      #           operator: In
      #           values: 
      #           - "dgis04"
      #           - "dgis05"
      #           - "dgis06"
      #           - "dgis07"
      #           - "dgis08"
      #           - "dgis09"
      containers:
      - name: redis
        image: redis:6.0
        ports:
        - containerPort: 6379
          name: client
        - containerPort: 16379
          name: gossip
        command: ["/conf/update-node.sh", "redis-server", "/conf/redis.conf"]
        args: ["--requirepass", "$(REDIS_PASS)"]
        env:
        - name: POD_IP
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
        - name: REDIS_PASS
          valueFrom:
            secretKeyRef:
              name: redis
              key: REDIS_PASS
        volumeMounts:
        - name: conf
          mountPath: /conf
          readOnly: false
        - name: redis-data
          mountPath: /data
          readOnly: false
      initContainers:
        - image: busybox
          command:
          - "sh"
          - "-c"
          - |
            # example: modify the sysctl
            echo 32678 > /proc/sys/net/core/somaxconn
            echo 1 > /proc/sys/vm/overcommit_memory
          imagePullPolicy: Never
          name: setsysctl
          securityContext:
            privileged: true
      volumes:
      - name: conf
        configMap:
          name: redis-cluster
          defaultMode: 0755
  volumeClaimTemplates:
  - metadata:
      name: redis-data
    spec:
      accessModes:
        - "ReadWriteOnce"
      resources:
        requests:
          storage: "500Gi"
      storageClassName: rook-ceph-block
~~~



## 四、Redis cluster proxy

官方地址：https://github.com/RedisLabs/redis-cluster-proxy

### 1、特性

- 路由：每个查询都会自动路由到集群的正确节点
- 多线程
- 支持多路复用和专用连接模型
- 即使在多路复用上下文中也能保证查询执行和回复顺序
- 错误后集群配置的自动更新`ASK|MOVED`：当回复中出现此类错误时，代理会通过获取集群的更新配置并重新映射所有插槽来自动更新集群的内部表示。所有查询将在更新完成后重新执行，因此，从客户端的角度来看，一切都正常进行（客户端不会收到 ASK|MOVED 错误：他们将在更新后直接收到预期的回复集群配置已更新）。
- 跨槽/跨节点查询：支持许多涉及属于不同槽（甚至不同集群节点）的多个键的命令。这些命令会将查询拆分为多个查询，这些查询将被路由到不同的槽/节点。这些命令的回复处理是特定于命令的。某些命令（例如`MGET`）会将所有回复合并为一个回复。其他命令如`MSET`or`DEL`将汇总所有回复的结果。由于这些查询实际上破坏了命令的原子性，因此它们的使用是可选的（默认情况下禁用）。有关更多信息，请参见下文。
- 一些没有特定节点/槽的命令`DBSIZE`被传送到所有节点，并且回复将被映射减少，以便给出所有回复中包含的所有值的总和。
- `PROXY`可用于执行某些特定于代理的操作的附加命令。

### 2、docker 单机搭建

​		可以手动打包，也可以使用docker hub上的镜像，本次进行手动打docker镜像的方式进行

* 安装编译

  ~~~shell
  # Make Install 
  git clone https://github.com/artix75/redis-cluster-proxy
  cd redis-cluster-proxy
  make PREFIX=/usr/local/redis_cluster_proxy install 
  ~~~

  经历如下报错

  * `proxy.h:61:5: error: unknown type name ‘_Atomic’`

    参考: https://github.com/RedisLabs/redis-cluster-proxy/issues/11

    ~~~
    yum install devtoolset-3-gcc devtoolset-3-gcc-c++   
    scl enable devtoolset-3 bash
    ~~~

  * `No package devtoolset-3-gcc available.No package devtoolset-3-gcc-c++ available.`

    No package devtoolset-3-gcc available
    添加 yum 源

    ~~~
    [copr:copr.fedorainfracloud.org:rhscl:devtoolset-3]
    name=Copr repo for devtoolset-3 owned by rhscl
    baseurl=https://download.copr.fedorainfracloud.org/results/rhscl/devtoolset-3/epel-6-$basearch/
    type=rpm-md
    skip_if_unavailable=True
    gpgcheck=1
    gpgkey=https://download.copr.fedorainfracloud.org/results/rhscl/devtoolset-3/pubkey.gpg
    repo_gpgcheck=0
    enabled=1
    enabled_metadata=1
    ~~~

  * `xorg-x11-server-utils-7.7-20.el7.x86_64 has missing requires of libXxf86vm.so.1()(64bit)`

    本机安装至此发现存在包冲突，在不影响其他应用的前提下，放弃了本次安装，使用 docker hub已有的镜像进行，有需要可以自行验证以下步骤

* dockerfile

  ~~~txt
  FROM centos:7
  WORKDIR /data
  ADD redis-cluster-proxy /usr/local/bin/
  EXPOSE 7777
  ~~~

* 打包

  ~~~sh
  docker build . -t redis-cluster-proxy:v1.0.0
  ~~~

* 创建 proxy.conf

  ~~~txt
      cluster redis-cluster-0.redis-cluster.default:6379,redis-cluster-1.redis-cluster.default:6379,redis-cluster-2.redis-cluster.default:6379,redis-cluster-3.redis-cluster.default:6379,redis-clust    er-4.redis-cluster.default:6379,redis-cluster-5.redis-cluster.default:6379     # 配置为Redis Cluster Service
      bind 0.0.0.0
      port 7777   # redis-cluster-proxy 对外暴露端口
      threads 8   # 线程数量
      daemonize no  
      enable-cross-slot yes    
      auth xxx     # 配置Redis Cluster 认证密码  
      logfile "/usr/local/redis-cluster-proxy/redis-cluster-proxy.log"
      log-level error
  ~~~

* 运行

  ~~~
   docker run -tid --name redis-proxy -p 7777:7777 -v /opt/software/redis-proxy:/data/redis-proxy kornrunner/redis-cluster-proxy:latest sh -c "redis-cluster-proxy -c /data/redis-proxy/proxy.conf"
  ~~~

* 链接测试

  ~~~sh
  redis-cli -h localhost -p 7777 -a xxx
  ~~~

  ~~~systemverilog
  root@redis-cluster-0:/data# redis-cli -h localhost -p 30001 -a dgisserver@redis4522
  Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
  localhost:30001> set 'a'
  (error) ERR wrong number of arguments for 'set' command
  localhost:30001> set a a
  OK
  localhost:30001> get a
  "a"
  localhost:30001> set aaddd a
  OK
  localhost:30001> set adada aa
  OK
  localhost:30001> 
  ~~~

### 3.k8s 搭建

* 下载镜像

  ~~~sh
  docker pull huanke/redis-cluster-proxy:v1.0.0
  docker tag huanke/redis-cluster-proxy:v1.0.0 redis-cluster-proxy:v1.0.0 
  ~~~

* 配置文件

  configmap

  ~~~ yaml
  ---
  # Redis-Proxy Config
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: redis-proxy
    namespace: default
  data:
    proxy.conf: |
      cluster xxx # 配置为Redis Cluster Service
      bind 0.0.0.0
      port 7777   # redis-cluster-proxy 对外暴露端口
      threads 8   # 线程数量
      daemonize no  
      enable-cross-slot yes    
      auth dgisserver@redis4522     # 配置Redis Cluster 认证密码  
      logfile "/usr/local/redis-cluster-proxy/redis-cluster-proxy.log"
      log-level error
  ~~~

  deployment

  service配置可以根据需要自定义配置

  ~~~yaml
  ---
  # Redis-Proxy NodePort
  apiVersion: v1
  kind: Service
  metadata:
    name: redis-proxy
    namespace: default
  spec:
    type: NodePort # 对K8S外部提供服务
    ports:
    - name: redis-proxy
      nodePort: 30001   # 对外提供的端口
      port: 7777
      protocol: TCP
      targetPort: 7777
    selector:
      app: redis-proxy
  ---
  # Redis-Proxy Deployment
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: redis-proxy
    namespace: default
  spec:
    replicas: 1
    selector:
      matchLabels:
        app: redis-proxy
    template:
      metadata:
        labels:
          app: redis-proxy
      spec:
        imagePullSecrets:
          - name: harbor
        containers:
          - name: redis-proxy
            image: redis-cluster-proxy:v1.0.0
            command: ["redis-cluster-proxy"]
            args:
              - -c
              - /data/proxy.conf   # 指定启动配置文件
            ports:
              - name: redis-7777
                containerPort: 7777
                protocol: TCP
            volumeMounts:
              - name: redis-proxy-conf
                mountPath: /data/
        volumes:   # 挂载proxy配置文件
          - name: redis-proxy-conf
            configMap:
              name: redis-proxy
  ~~~

  验证如下

  ~~~systemverilog
  root@redis-cluster-0:/data# redis-cli redis-proxy 30001 -a dgisserver@redis4522
  (error) ERR unknown command `redis-proxy`, with args beginning with: `30001`, `-a`, `dgisserver@redis4522`, 
  root@redis-cluster-0:/data# ping redis-proxy
  bash: ping: command not found
  root@redis-cluster-0:/data# redis-cli -h redis-proxy -p 30001 -a dgisserver@redis4522
  Warning: Using a password with '-a' or '-u' option on the command line interface may not be safe.
  redis-proxy:30001> set 'a'
  (error) ERR wrong number of arguments for 'set' command
  redis-proxy:30001> set a a
  OK
  redis-proxy:30001> get a
  "a"
  redis-proxy:30001> set aaddd a
  OK
  redis-proxy:30001> set adada aa
  OK
  redis-proxy:30001> 
  ~~~

  