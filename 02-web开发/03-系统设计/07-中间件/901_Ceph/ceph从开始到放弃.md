## 1. Ceph简介

​	Ceph是一个统一的分布式存储系统，提供较好的性能、可靠性、可拓展性

​	支持块存储、对象存储、文件系统挂载

[ceph官网](https://docs.ceph.com/en/quincy/)



Monitors：Ceph Monitor ( ceph-mon) 维护集群状态的映射，包括监视器映射、管理器映射、OSD 映射、MDS 映射和 CRUSH 映射。这些映射是 Ceph 守护进程相互协调所需的关键集群状态。监视器还负责管理守护进程和客户端之间的身份验证。冗余和高可用性通常需要至少三个监视器。

Managers：Ceph Manager守护进程 ( ceph-mgr) 负责跟踪运行时指标和 Ceph 集群的当前状态，包括存储利用率、当前性能指标和系统负载。Ceph Manager 守护进程还托管基于 python 的模块来管理和公开 Ceph 集群信息，包括基于 Web 的Ceph Dashboard和 REST API。高可用性通常需要至少两个管理器。

Ceph OSD：一个Ceph OSD（对象存储守护进程 ceph-osd）存储数据、处理数据复制、恢复、重新平衡，并通过检查其他 Ceph OSD 守护进程的心跳来向 Ceph 监视器和管理器提供一些监视信息。冗余和高可用性通常需要至少三个 Ceph OSD。

MDS：Ceph 元数据服务器(MDS ceph-mds) 代表Ceph 文件系统存储元数据（即 Ceph 块设备和 Ceph 对象存储不使用 MDS）。Ceph 元数据服务器允许 POSIX 文件系统用户执行基本命令（如 ls、find等），而不会给 Ceph 存储集群带来巨大负担。
