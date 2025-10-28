# Docker 与 自定义 iptables 冲突问题

## 一. 分析无法访问docker容器端口问题

~~~
作为一个熟练掌握linux、docker、iptables的高级运维工程师，帮我分析一下下面的问题
问题现象：
使用docker运行的容器端口，运行 telent 127.0.0.1 18443 可以正常连接，使用 telent 主网卡ip 18443 连接被拒绝。
系统是centos, 关闭了firewalld，使用iptables。
初步的解决方法 ，重启docker，然后 telent 主网卡ip 18443 可以成功。
但是将iptables规则重新应用之后，又不能连接。这时候重启docker，然后又可以成功了。
我觉得可能原因是 iptables 配置的规则与docker 存在冲突。你帮我分析一下原因呢。
~~~

腾讯元宝输出

你的分析非常准确，这确实是 iptables 规则与 Docker 的冲突问题。让我从技术原理层面详细分析一下。

### 1. 问题根本原因

#### Docker 的 iptables 管理机制

1. **Docker 自动管理 iptables**：当 Docker 服务启动时，它会在 `DOCKER-USER`和 `DOCKER`链中自动创建规则，用于容器端口转发和网络隔离。

2. **重启 Docker 的影响**：

   ```
   systemctl restart docker
   ```

   重启 Docker 会：

   - 清除所有 Docker 相关的 iptables 规则
   - 重新创建正确的端口转发规则
   - 这就是为什么重启后能正常访问的原因

3. **iptables 规则重新应用的冲突**： 当你执行 `iptables-restore`或重新应用自定义规则时，可能会： 覆盖或删除 Docker 创建的规则 改变规则链的顺序 导致容器端口转发失效

#### 具体的技术细节

#### Docker 的正常规则结构

```
#### 查看完整的 iptables 规则
iptables -t nat -L -n -v
iptables -t filter -L -n -v

#### Docker 通常会创建这样的规则：
#### NAT 表：
Chain PREROUTING (policy ACCEPT)
target     prot opt source               destination         
DOCKER     all  --  0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL

Chain DOCKER (2 references)
target     prot opt source               destination         
RETURN     all  --  0.0.0.0/0            0.0.0.0/0           
DNAT       tcp  --  0.0.0.0/0            0.0.0.0/0            tcp dpt:18443 to:172.17.0.2:443

#### Filter 表：
Chain FORWARD (policy DROP)
target     prot opt source               destination         
DOCKER-USER  all  --  0.0.0.0/0            0.0.0.0/0           
DOCKER-ISOLATION-STAGE-1  all  --  0.0.0.0/0            0.0.0.0/0           
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0            
ACCEPT     all  --  0.0.0.0/0            0.0.0.0/0
```

### 2. 解决方案

#### 方案1：在 DOCKER-USER 链中添加规则（推荐）

Docker 官方推荐在 `DOCKER-USER`链中添加自定义规则，因为这个链在 Docker 规则之前处理，且不会被 Docker 覆盖。

```
#### 添加你的自定义规则到 DOCKER-USER 链
iptables -I DOCKER-USER -i eth0 -p tcp --dport 18443 -j ACCEPT

#### 或者更精确的规则
iptables -I DOCKER-USER -s 0.0.0.0/0 -d 你的服务器IP -p tcp --dport 18443 -j ACCEPT
```

#### 方案2：修改 iptables 规则保存/恢复脚本

在保存和恢复 iptables 规则时，排除或保留 Docker 相关规则：

```
#### 保存规则时排除 Docker 相关链
iptables-save | grep -v DOCKER > /etc/sysconfig/iptables

#### 或者只保存非 Docker 规则
iptables-save | awk '/^:*DOCKER/ {next} {print}' > /etc/sysconfig/iptables.no-docker
```

#### 方案3：使用 Docker 的配置文件管理网络

在 `/etc/docker/daemon.json`中配置：

```
{
  "iptables": false,
  "userland-proxy": false
}
```

然后手动管理 iptables 规则（不推荐，除非你有特殊需求）。

#### 方案4：创建专业的规则管理脚本

```
#!/bin/bash
#### restore-iptables.sh

#### 停止 Docker 服务
systemctl stop docker

#### 清除所有规则
iptables -F
iptables -t nat -F
iptables -t mangle -F

#### 恢复你的自定义规则
iptables-restore < /path/to/your/iptables-backup

#### 启动 Docker 服务（Docker 会自动添加所需规则）
systemctl start docker

#### 在 DOCKER-USER 链中添加必要的规则
iptables -I DOCKER-USER -p tcp --dport 18443 -j ACCEPT
```

### 3. 诊断命令

检查当前规则状态：

```
#### 检查 NAT 表规则
iptables -t nat -L -n -v --line-numbers

#### 检查 Filter 表规则
iptables -L -n -v --line-numbers

#### 检查具体的端口转发
iptables -t nat -L DOCKER -n -v

#### 跟踪数据包路径（诊断连接问题）
iptables -t raw -A OUTPUT -p tcp --dport 18443 -j TRACE
iptables -t raw -A PREROUTING -p tcp --dport 18443 -j TRACE
```

### 4. 最佳实践建议

1. **规则顺序管理**：始终在 `DOCKER-USER`链中添加自定义规则
2. **规则备份策略**：备份时区分 Docker 规则和自定义规则
3. **服务启动顺序**：先启动 Docker，再应用自定义 iptables 规则
4. **监控规则变化**：使用 `iptables-save`定期检查规则完整性

你的问题很典型，正是由于 iptables 规则恢复时破坏了 Docker 创建的端口转发规则链。采用方案1（使用 DOCKER-USER 链）应该能彻底解决问题。

## 二、Docker与Iptables



### 1. Docker 的 iptables 架构原理

#### Docker 的自动规则管理

当 Docker 守护进程启动时，它会自动创建和管理一套复杂的 iptables 规则：

```
### 查看完整的 iptables 规则结构
iptables-save | grep -E "(DOCKER|FORWARD|POSTROUTING)"
```

#### Docker 创建的主要规则链

##### **Filter 表：**

- `DOCKER`：处理容器端口转发规则
- `DOCKER-USER`：用户自定义规则的位置（不会被 Docker 覆盖）
- `DOCKER-ISOLATION`：容器网络隔离规则

##### **NAT 表：**

- `DOCKER`：处理 DNAT 端口映射

### 2. 详细规则分析

#### 查看完整的 Docker 规则

```
### 查看 NAT 表的完整规则（端口转发的核心）
iptables -t nat -L -n -v --line-numbers

# 查看 Filter 表的 Docker 相关规则
iptables -L -n -v --line-numbers | grep -A10 -B10 DOCKER

# 查看详细的规则结构
iptables-save | grep -A20 -B20 DOCKER
```

#### 典型规则示例分析

假设运行了一个 Nginx 容器：

```
docker run -d -p 80:80 --name nginx nginx
```

Docker 会创建如下规则：

**NAT 表规则：**

```
# PREROUTING 链 - 入口流量DNAT
-A PREROUTING -m addrtype --dst-type LOCAL -j DOCKER
-A DOCKER -i docker0 -p tcp -m tcp --dport 80 -j DNAT --to-destination 172.17.0.2:80

# OUTPUT 链 - 本地发出的流量DNAT  
-A OUTPUT -m addrtype --dst-type LOCAL -j DOCKER
-A DOCKER -o docker0 -p tcp -m tcp --dport 80 -j DNAT --to-destination 172.17.0.2:80

# POSTROUTING 链 - SNAT（MASQUERADE）
-A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
```

**Filter 表规则：**

```
# FORWARD 链 - 容器流量转发
-A FORWARD -j DOCKER-USER
-A FORWARD -j DOCKER-ISOLATION-STAGE-1
-A FORWARD -o docker0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
-A FORWARD -o docker0 -j DOCKER
-A FORWARD -i docker0 ! -o docker0 -j ACCEPT
-A FORWARD -i docker0 -o docker0 -j ACCEPT

# DOCKER 链 - 具体的端口放行规则
-A DOCKER -d 172.17.0.2/32 ! -i docker0 -o docker0 -p tcp -m tcp --dport 80 -j ACCEPT
```

### 3. Docker 网络模式与 iptables

#### 不同网络模式的 iptables 影响

**bridge 模式（默认）：**

```
# 创建自定义bridge网络
docker network create --driver bridge mynet

# 查看该网络的iptables规则
iptables -L | grep -A5 -B5 mynet
```

**host 模式：**

```
docker run --network host nginx
# 不会创建Docker NAT规则，直接使用主机网络栈
```

**none 模式：**

```
docker run --network none nginx
# 无网络连接，不涉及iptables规则
```

### 4. 常见问题与解决方案

#### 问题1：Docker 规则被自定义规则覆盖

**错误做法：**

```
# 清空所有规则（会破坏Docker的网络）
iptables -F
iptables -t nat -F
```

**正确做法：**

```
# 只清空非Docker规则
iptables-save | grep -v DOCKER | iptables-restore

# 或者重启Docker服务来重建规则
systemctl restart docker
```

#### 问题2：自定义防火墙规则与 Docker 冲突

**解决方案：使用 DOCKER-USER 链**

```
# 错误：直接添加到INPUT链（对容器流量无效）
iptables -A INPUT -p tcp --dport 80 -j DROP

# 正确：添加到DOCKER-USER链
iptables -A DOCKER-USER -p tcp --dport 80 -j DROP

# 允许特定IP访问容器
iptables -I DOCKER-USER -s 192.168.1.0/24 -j ACCEPT
iptables -I DOCKER-USER -j DROP  # 默认拒绝其他访问
```

#### 问题3：容器间网络隔离

**启用容器间网络隔离：**

```
# 创建网络时启用隔离
docker network create --driver bridge --opt com.docker.network.bridge.enable_icc=false isolated_net

# 或者通过iptables手动隔离
iptables -I DOCKER-USER -i docker0 -o docker0 -j DROP
```

### 5. 高级配置技巧

#### 禁用 Docker 的 iptables 管理

#### **不推荐，除非你有特殊需求：**

```
// /etc/docker/daemon.json
{
  "iptables": false,
  "userland-proxy": false
}
```

禁用后需要手动管理所有规则：

```
# 手动创建Docker网络规则
iptables -t nat -A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
iptables -A FORWARD -i docker0 -o eth0 -j ACCEPT
iptables -A FORWARD -i eth0 -o docker0 -m state --state RELATED,ESTABLISHED -j ACCEPT
```

#### 使用 ipset 优化大量规则

```
# 创建ipset集合
ipset create allowed_ips hash:ip
ipset add allowed_ips 192.168.1.100
ipset add allowed_ips 10.0.0.0/24

# 在DOCKER-USER链中使用ipset
iptables -I DOCKER-USER -m set --match-set allowed_ips src -j ACCEPT
iptables -A DOCKER-USER -j DROP
```

#### 动态防火墙规则管理

```
#!/bin/bash
# docker-firewall-manager.sh

# 备份当前规则
iptables-save > /tmp/iptables.backup.$(date +%s)

# 清理旧规则（保留DOCKER相关规则）
iptables-save | grep -v -E "(DOCKER|DOCKER-USER)" | iptables-restore

# 在DOCKER-USER链中应用新规则
iptables -F DOCKER-USER

# 允许管理网络
iptables -I DOCKER-USER -s 10.0.0.0/8 -j ACCEPT
iptables -I DOCKER-USER -s 192.168.0.0/16 -j ACCEPT

# 允许特定端口
for port in 80 443 22; do
    iptables -I DOCKER-USER -p tcp --dport $port -j ACCEPT
done

# 默认拒绝
iptables -A DOCKER-USER -j DROP

echo "Docker防火墙规则更新完成"
```

### 6. 故障排查命令

#### 检查规则顺序和优先级

```
# 查看规则顺序（带行号）
iptables -L DOCKER-USER -n --line-numbers

# 检查规则匹配计数
iptables -L DOCKER-USER -n -v

# 跟踪数据包路径
iptables -t raw -A PREROUTING -p tcp --dport 80 -j TRACE
# 查看日志：dmesg | tail -20
```

#### 容器网络连通性测试

```
# 进入容器测试网络
docker exec -it nginx bash

# 在容器内测试
curl -I http://172.17.0.1:80  # 宿主机docker0接口
ping 8.8.8.8

# 从外部测试容器端口
telnet 宿主机IP 80
nc -zv 宿主机IP 80
```

#### Docker 网络诊断

```
# 查看Docker网络详情
docker network ls
docker network inspect bridge

# 查看容器网络配置
docker inspect nginx | grep -A 30 "NetworkSettings"

# 检查Docker守护进程日志
journalctl -u docker -f
```

### 7. 安全最佳实践

#### 1. 最小权限原则

```
# 只暴露必要的端口
docker run -p 80:80 nginx  # 而不是 -p 0.0.0.0:80:80

# 使用内部网络通信
docker network create internal
docker run --network internal --name app app-image
docker run --network internal --name db db-image
```

#### 2. 网络分段

```
# 为不同环境创建不同网络
docker network create frontend
docker network create backend
docker network create database
```

#### 3. 监控和审计

```
# 监控Docker相关iptables规则变化
iptables-save > /tmp/current_rules
diff /tmp/current_rules /tmp/previous_rules

# 记录异常连接尝试
iptables -I DOCKER-USER -p tcp --dport 22 -m limit --limit 3/min -j LOG --log-prefix "SSH_BRUTE_FORCE: "
```

### 8. 与 Kubernetes 的对比

如果是 Kubernetes 环境，情况更复杂：

```
# Kubernetes使用更复杂的规则结构
iptables-save | grep -E "(KUBE|KUBE-SERVICES)"

# 主要由kube-proxy管理规则
# 建议使用calico、flannel等CNI插件
```

### 9. 总结

Docker 与 iptables 的集成是一个强大的功能，但需要深入理解其工作原理：

1. **规则优先级**：DOCKER-USER > DOCKER-ISOLATION > DOCKER
2. **流量路径**：PREROUTING → FORWARD → POSTROUTING
3. **安全原则**：最小权限、网络分段、审计监控
4. **故障排查**：规则顺序、流量跟踪、日志分析

## 三、请求Docker容器时转发流程

好的，我来详细解析请求 Docker 容器端口时的完整 iptables 转发流程。这是一个典型的数据包生命周期分析。

### 1. 实验环境设定

假设我们有以下环境：

- **宿主机 IP**: 192.168.1.100
- **Docker 网络**: 172.17.0.0/16
- **容器 IP**: 172.17.0.2
- **暴露端口**: `docker run -p 80:80 nginx`

外部客户端访问：`http://192.168.1.100:80`

### 2. 完整转发流程分析

#### 阶段1：数据包到达宿主机（PREROUTING）

**外部客户端发送数据包：**

```
源IP: 客户端IP, 源端口: 随机端口
目标IP: 192.168.1.100, 目标端口: 80
```

**iptables 处理流程：**

```
# 查看 PREROUTING 链规则
iptables -t nat -L PREROUTING -n -v --line-numbers

# 典型规则：
Chain PREROUTING (policy ACCEPT)
num   pkts bytes target     prot opt in     out     source               destination         
1       10   600 DOCKER     all  --  *      *       0.0.0.0/0            0.0.0.0/0            ADDRTYPE match dst-type LOCAL
```

**数据包匹配过程：**

1. **进入 PREROUTING 链**
2. **匹配规则 1**: `ADDRTYPE match dst-type LOCAL` 检查目标地址是否是本地地址（192.168.1.100 是宿主机IP，属于LOCAL） **动作**: 跳转到 `DOCKER`链

#### 阶段2：DOCKER NAT 链处理（DNAT）

```
# 查看 DOCKER 链规则
iptables -t nat -L DOCKER -n -v --line-numbers

# 典型规则：
Chain DOCKER (2 references)
num   pkts bytes target     prot opt in     out     source               destination         
1        5   300 RETURN     all  --  docker0 *       0.0.0.0/0            0.0.0.0/0           
2        5   300 DNAT       tcp  --  !docker0 *       0.0.0.0/0            0.0.0.0/0            tcp dpt:80 to:172.17.0.2:80
```

**DNAT 转换过程：**

1. **进入 DOCKER 链**
2. **检查规则 1**: 输入接口是 docker0？→ 否（数据包来自 eth0）
3. **检查规则 2**: 输入接口不是 docker0 ✅ 协议是 TCP ✅ 目标端口是 80 ✅ **执行 DNAT**: `to:172.17.0.2:80`

**DNAT 转换结果：**

```
转换前: 目标IP: 192.168.1.100:80
转换后: 目标IP: 172.17.0.2:80
```

#### 阶段3：路由决策

经过 DNAT 后，内核进行路由查询：

```
# 查看路由表
route -n

# 典型输出：
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
0.0.0.0         192.168.1.1     0.0.0.0         UG    100    0        0 eth0
172.17.0.0      0.0.0.0         255.255.0.0     U     0      0        0 docker0
192.168.1.0     0.0.0.0         255.255.255.0   U     100    0        0 eth0
```

**路由判断：**

- 目标 IP: 172.17.0.2
- 匹配路由: 172.17.0.0/16 → 接口 docker0
- **结论**: 需要转发到 docker0 接口（不是本地进程）

#### 阶段4：FORWARD 链过滤

数据包进入 FORWARD 链，这是防火墙过滤的核心：

```
# 查看 FORWARD 链规则
iptables -L FORWARD -n -v --line-numbers

# 典型规则结构：
Chain FORWARD (policy DROP)
num   pkts bytes target     prot opt in     out     source               destination         
1     1000 50000 DOCKER-USER  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
2     1000 50000 DOCKER-ISOLATION-STAGE-1  all  --  *      *       0.0.0.0/0            0.0.0.0/0           
3     5000 250K ACCEPT     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0            ctstate RELATED,ESTABLISHED
4       10   600 DOCKER     all  --  *      docker0  0.0.0.0/0            0.0.0.0/0           
5     2000 100K ACCEPT     all  --  docker0 !docker0  0.0.0.0/0            0.0.0.0/0           
6     1000 50000 ACCEPT     all  --  docker0 docker0  0.0.0.0/0            0.0.0.0/0
```

**详细转发流程：**

##### 步骤 4.1: DOCKER-USER 链（用户自定义规则）

```
# 查看 DOCKER-USER 链
iptables -L DOCKER-USER -n -v --line-numbers

# 如果没有自定义规则，直接返回FORWARD链
```

##### 步骤 4.2: DOCKER-ISOLATION-STAGE-1 链

```
# 容器网络隔离检查
iptables -L DOCKER-ISOLATION-STAGE-1 -n -v

# 典型规则：
Chain DOCKER-ISOLATION-STAGE-1 (1 references)
target     prot opt source               destination         
DOCKER-ISOLATION-STAGE-2  all  --  0.0.0.0/0            0.0.0.0/0           
RETURN     all  --  0.0.0.0/0            0.0.0.0/0
```

##### 步骤 4.3: 连接状态检查

- **规则 3**: `ACCEPT all -- * docker0 ctstate RELATED,ESTABLISHED` 检查是否是已建立的连接（对于新连接不匹配） 输入接口: any, 输出接口: docker0 **我们的数据包**: 新连接，不匹配此规则

##### 步骤 4.4: DOCKER 链过滤检查

```
# 查看 FILTER 表的 DOCKER 链
iptables -L DOCKER -n -v --line-numbers

# 典型规则：
Chain DOCKER (1 references)
target     prot opt in     out     source               destination         
ACCEPT     tcp  --  !docker0 docker0  0.0.0.0/0            172.17.0.2           tcp dpt:80
```

**关键匹配：**

- 输入接口: 不是 docker0 ✅（来自 eth0）
- 输出接口: docker0 ✅（目标容器在 docker0）
- 目标地址: 172.17.0.2 ✅
- 目标端口: 80 ✅
- **动作**: ACCEPT ✅

#### 阶段5：POSTROUTING 链（SNAT/MASQUERADE）

数据包即将发送到容器前的最后处理：

```
# 查看 POSTROUTING 链
iptables -t nat -L POSTROUTING -n -v --line-numbers

# 典型规则：
Chain POSTROUTING (policy ACCEPT)
target     prot opt in     out     source               destination         
MASQUERADE  all  --  *      !docker0  172.17.0.0/16        0.0.0.0/0           
MASQUERADE  all  --  *      !docker0  172.18.0.0/16        0.0.0.0/0
```

**SNAT 处理：**

- 源地址: 客户端IP
- 目标地址: 172.17.0.2
- **匹配规则**: 源网络 172.17.0.0/16，输出接口不是 docker0
- **执行 MASQUERADE**: 将源地址替换为宿主机出口IP

**SNAT 转换结果：**

```
转换前: 源IP: 客户端IP
转换后: 源IP: 192.168.1.100（宿主机IP）
```

#### 阶段6：数据包到达容器

经过完整的 iptables 处理，数据包最终到达容器：

- 目标地址: 172.17.0.2:80（容器内的 nginx）
- 源地址: 192.168.1.100（宿主机，经过 MASQUERADE）

#### 响应数据包的回程路径

#### 阶段7：容器响应处理

**容器发出响应：**

```
源IP: 172.17.0.2:80, 目标IP: 192.168.1.100
```

#### 阶段8：响应包的 NAT 处理（反向）

**POSTROUTING 链：**

- 源地址: 172.17.0.2
- **匹配 MASQUERADE 规则**（但这次输出接口是 eth0，不是 docker0）
- 执行 SNAT: 源地址改为 192.168.1.100

**PREROUTING 链：**

- 目标地址是客户端IP，不是LOCAL地址，跳过DOCKER链

**FORWARD 链：**

- **匹配规则 3**: `ACCEPT all -- * docker0 ctstate RELATED,ESTABLISHED`
- 这是已建立的连接响应，直接ACCEPT

### 3. 完整流程图解

```
外部客户端请求:
    src: ClientIP:random_port → dst: 192.168.1.100:80
        ↓
宿主机 PREROUTING (nat):
    DOCKER链 DNAT: dst → 172.17.0.2:80
        ↓
路由决策: 目标 172.17.0.2 → 转发到 docker0
        ↓
FORWARD链 (filter):
    DOCKER-USER → DOCKER-ISOLATION → DOCKER链 ACCEPT
        ↓
POSTROUTING (nat):
    MASQUERADE: src → 192.168.1.100
        ↓
到达容器: src: ClientIP → dst: 172.17.0.2:80
        ↓
容器响应: src: 172.17.0.2:80 → dst: ClientIP
        ↓
POSTROUTING: MASQUERADE (反向)
        ↓
返回客户端
```

### 4. 调试和验证命令

#### 实时跟踪数据包

```
# 添加跟踪规则
iptables -t raw -I PREROUTING -p tcp --dport 80 -j TRACE
iptables -t raw -I OUTPUT -p tcp --dport 80 -j TRACE

# 查看跟踪日志
dmesg | grep -i trace

# 清理跟踪规则
iptables -t raw -F
```

#### 规则匹配统计

```
# 查看规则匹配计数
iptables -L -n -v
iptables -t nat -L -n -v

# 重置计数器
iptables -Z
iptables -t nat -Z
```

#### 连接跟踪查看

```
# 查看当前连接跟踪
conntrack -L | grep 172.17.0.2

# 查看NAT转换表
cat /proc/net/nf_conntrack | grep 172.17.0.2
```



