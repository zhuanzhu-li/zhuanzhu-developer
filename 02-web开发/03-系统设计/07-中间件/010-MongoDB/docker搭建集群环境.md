[toc]

# MongoDB Sharding集群搭建

参考：https://blog.csdn.net/bxg_kyjgs/article/details/125784629


## 1. 环境准备

### 1.1 docker 环境搭建

### 1.2 网络构建

### 1.3 集群架构

整体架构
整体架构涉及到15个节点，我们这里使用Docker容器进行部署

那么我们先来总结一下我们搭建一个高可用集群需要多少个Mongo

mongos： 3台

configserver ： 3台

shard ： 3片； 每个分片由三个节点构成

| 角色           | 端口  | 暴漏端口 | 描述       | 角色     |
| -------------- | ----- | -------- | ---------- | -------- |
| config-server1 | 27017 | –        | 配置节点1  | –        |
| config-server2 | 27017 | –        | 配置节点2  | –        |
| config-server3 | 27017 | –        | 配置节点3  | –        |
| mongos-server1 | 27017 | 30001    | 路由节点1  | –        |
| mongos-server2 | 27017 | 30002    | 路由节点2  | –        |
| mongos-server3 | 27017 | 30003    | 路由节点3  | –        |
| shard1-server1 | 27017 | –        | 分片1节点1 | Primary  |
| shard1-server2 | 27017 | –        | 分片1节点2 | Secondry |
| shard1-server3 | 27017 | –        | 分片1节点3 | Arbiter  |
| shard2-server1 | 27017 | –        | 分片2节点1 | Primary  |
| shard2-server2 | 27017 | –        | 分片2节点2 | Secondry |
| shard2-server3 | 27017 | –        | 分片2节点3 | Arbiter  |
| shard3-server1 | 27017 | –        | 分片3节点1 | Primary  |
| shard3-server2 | 27017 | –        | 分片3节点2 | Secondry |
| shard3-server3 | 27017 | –        | 分片3节点3 | Arbiter  |

![MongoDB集群](../../99_source/img/MongoDB集群.png)