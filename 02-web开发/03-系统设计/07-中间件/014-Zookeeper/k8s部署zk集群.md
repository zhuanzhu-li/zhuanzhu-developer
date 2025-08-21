# Zookeeper集群部署

## 1、获取镜像

由于docker-hub 国内无法访问，可以通过 https://docker.aityp.com/ 镜像下载，然后重新tag

~~~shell
docker pull swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/zookeeper:3.9.3
docker tag swr.cn-north-4.myhuaweicloud.com/ddn-k8s/docker.io/zookeeper:3.9.3 zookeeper:3.9.3 
~~~



## 2、 K8s配置

~~~yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: zookeeper-config
data:
  zoo.cfg: |
    tickTime=5000
    initLimit=10
    syncLimit=5
    dataDir=/data
    clientPort=2181
    autopurge.snapRetainCount=3
    #zookeeper.electionPortBindRetry=10
    autopurge.purgeInterval=1
    server.1=zookeeper-cluster-0.zookeeper-cs.default.svc.cluster.local:2888:3888
    server.2=zookeeper-cluster-1.zookeeper-cs.default.svc.cluster.local:2888:3888
    server.3=zookeeper-cluster-2.zookeeper-cs.default.svc.cluster.local:2888:3888
    quorumListenOnAllIPs=true
---
apiVersion: v1
kind: Service
metadata:
  name: zookeeper-cs
  labels:
    app: zookeeper
spec:
  type: ClusterIP
  ports:
    - name: client
      port: 2181
      targetPort: 2181
    - name: quorum
      port: 2888
      targetPort: 2888
    - name: leader-election
      port: 3888
      targetPort: 3888
  selector:
    app: zookeeper

---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: zookeeper-cluster
  labels:
    app: zookeeper
spec:
  serviceName: zookeeper-cs
  replicas: 3
  selector:
    matchLabels:
      app: zookeeper
  template:
    metadata:
      labels:
        app: zookeeper
    spec:
      # 集群需要开启互斥
      # affinity:
      #   podAntiAffinity:
      #     requiredDuringSchedulingIgnoredDuringExecution:
      #       - labelSelector:
      #           matchExpressions:
      #             - key: "app"
      #               operator: In
      #               values:
      #               - zookeeper
      #         topologyKey: "kubernetes.io/hostname"
      containers:
        - name: zookeeper-cluster
          image: zookeeper:3.9.3
          imagePullPolicy: IfNotPresent
          command: ["sh", "-c"]
          args:
            - |
              # 从 Pod 名称截取序号（如 zk-0 → 0）
              ID=${HOSTNAME##*-}
              # 生成 myid（序号+1）
              echo $((ID + 1)) > /data/myid
              # 启动 ZooKeeper
              exec /docker-entrypoint.sh zkServer.sh start-foreground
          ports:
            - containerPort: 2181
              name: client
            - containerPort: 2888
              name: quorum
            - containerPort: 3888
              name: leader-election
          volumeMounts:
            - name: config
              mountPath: /conf   # 此目录是zk镜像的变量ZOOCFGDIR=/conf
      volumes:
        - name: config
          configMap:
            name: zookeeper-config

~~~

