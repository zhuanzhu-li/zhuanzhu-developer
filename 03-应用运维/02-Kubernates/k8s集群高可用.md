# k8s集群高可用

## 1、keepalived+haproxy

Keepalived + HAProxy 实现高可用集群的原理基于 VRRP（Virtual Router Redundancy Protocol，虚拟路由冗余协议）和负载均衡技术。以下是详细的实现原理：

### 1. **VRRP 协议原理**
VRRP 是一种用于提高网络冗余性和可靠性的协议，它允许多个物理路由器（或服务器）虚拟出一个“虚拟路由器”，并共享一个虚拟 IP 地址（VIP）。在 Keepalived 中，VRRP 被用来实现 VIP 的漂移和故障切换。

#### 关键概念
- **Virtual Router（虚拟路由器）**：
  - 由多个物理节点组成，对外表现为一个虚拟的路由器。
  - 每个虚拟路由器通过 `virtual_router_id` 来标识。
- **VIP（Virtual IP Address）**：
  - 虚拟 IP 地址，是客户端访问集群的入口地址。
  - 在正常情况下，VIP 绑定在主节点上；当主节点故障时，VIP 会漂移到备份节点。
- **Master 和 Backup 状态**：
  - **Master**：主节点，负责处理客户端请求，并持有 VIP。
  - **Backup**：备份节点，随时准备接管 VIP。
- **Priority（优先级）**：
  - 用于决定哪个节点成为 Master。优先级最高的节点将成为 Master。
  - 如果 Master 节点故障，优先级次高的 Backup 节点将成为新的 Master。

#### 工作流程
1. **初始化**：
   - 所有节点启动 Keepalived 服务后，根据配置文件中的 `state` 和 `priority`，节点会进入 Master 或 Backup 状态。
   - 如果有多个节点配置为 Master，优先级最高的节点将成为实际的 Master。
2. **VRRP 通告**：
   - Master 节点会定期发送 VRRP 通告（`advertise` 包）到本地网络的广播地址（如 224.0.0.18）。
   - Backup 节点监听这些通告包，以确认 Master 节点是否正常运行。
3. **故障检测**：
   - 如果 Backup 节点在一定时间内未收到 Master 节点的通告包（超时时间由 `advert_int` 和 `deadtime` 参数决定），它会认为 Master 节点故障。
   - 此时，优先级最高的 Backup 节点将提升为新的 Master，并接管 VIP。
4. **VIP 漂移**：
   - 新的 Master 节点会绑定 VIP，并发送 ARP 广播，通知网络中的其他设备更新 ARP 表，将 VIP 的 MAC 地址指向新的 Master 节点。
   - 客户端的流量将通过新的 Master 节点进行处理，从而实现故障切换。

### 2. **HAProxy 负载均衡原理**
HAProxy 是一个高性能的负载均衡器，用于将客户端的请求分发到后端的多个服务器上。它通常与 Keepalived 配合使用，提供高可用性和负载均衡功能。

#### 关键概念
- **Frontend（前端）**：
  - 定义了客户端访问的入口，通常绑定到 VIP。
  - 例如，`bind 192.168.1.100:80` 表示客户端通过 VIP 的 80 端口访问。
- **Backend（后端）**：
  - 定义了后端服务器的列表，HAProxy 会将请求转发到这些服务器。
  - 可以配置轮询、最少连接等负载均衡算法。
- **ACL（访问控制列表）**：
  - 用于定义请求的匹配规则，可以根据请求的特征（如 URL、源 IP 等）将请求转发到不同的后端。

#### 工作流程
1. **客户端请求**：
   - 客户端通过 VIP 发起请求（如 HTTP 请求）。
   - Keepalived 确保 VIP 始终绑定在可用的 Master 节点上。
2. **HAProxy 接收请求**：
   - HAProxy 在 Master 节点上监听 VIP 的端口，接收客户端的请求。
3. **请求分发**：
   - HAProxy 根据配置的负载均衡算法（如轮询、最少连接等），将请求分发到后端的服务器。
   - 例如，配置如下：
     ```plaintext
     frontend http_front
         bind 192.168.1.100:80
         default_backend http_back
     
     backend http_back
         balance roundrobin
         server server1 192.168.1.101:80 check
         server server2 192.168.1.102:80 check
     ```
4. **后端服务器处理**：
   - 后端服务器接收 HAProxy 转发的请求，并返回响应。
   - HAProxy 将后端服务器的响应转发回客户端。

### 3. **Keepalived + HAProxy 的高可用实现**
Keepalived 和 HAProxy 的结合使用，实现了高可用性和负载均衡的双重功能：

#### 高可用性
- **VIP 漂移**：
  - Keepalived 使用 VRRP 协议管理 VIP，确保在主节点故障时，VIP 可以快速漂移到备份节点。
  - 客户端始终通过 VIP 访问服务，无需关心后端节点的变化。
- **故障检测**：
  - Keepalived 通过 VRRP 通告机制快速检测主节点故障，并触发故障切换。
  - HAProxy 可以配置健康检查机制，检测后端服务器的可用性，并自动移除故障的后端服务器。

#### 负载均衡
- **请求分发**：
  - HAProxy 负责将客户端的请求分发到后端的多个服务器，提高系统的吞吐量和性能。
  - 可以根据不同的负载均衡算法（如轮询、最少连接等）进行配置。
- **会话保持（可选）**：
  - HAProxy 可以配置会话保持机制，确保同一客户端的请求始终被转发到同一后端服务器。

### 4. **总结**
Keepalived + HAProxy 的高可用集群实现原理如下：
- **Keepalived**：
  - 使用 VRRP 协议管理 VIP，确保 VIP 始终绑定在可用的节点上。
  - 通过 VRRP 通告机制检测故障，并触发 VIP 漂移。
- **HAProxy**：
  - 负责接收客户端请求，并将其分发到后端的多个服务器。
  - 提供负载均衡功能，提高系统的吞吐量和性能。
  - 可以配置健康检查机制，确保后端服务器的可用性。

这种组合不仅提高了系统的可靠性，还通过负载均衡提高了资源利用率和性能。

是的，`virtual_router_id` 在 Keepalived 配置中必须保持一致，这是非常关键的配置项。

### 5. `virtual_router_id` 的作用
`virtual_router_id` 是 VRRP（虚拟路由冗余协议）中用于标识虚拟路由器的唯一标识符。在 Keepalived 的 VRRP 实现中，所有参与同一 VIP 管理的节点必须使用相同的 `virtual_router_id`，否则它们无法识别彼此属于同一个虚拟路由器组，也就无法正确进行 VIP 的漂移和状态同步。

#### 配置示例
假设你的集群中有三个节点，主节点和两个备份节点，它们的 Keepalived 配置文件中 `virtual_router_id` 必须一致。以下是示例配置：

#####  主节点配置
```plaintext
vrrp_instance VI_1 {
    state MASTER
    interface bond0
    virtual_router_id 51
    priority 100
    advert_int 1
    virtual_ipaddress {
        192.168.1.100
    }
}
```

##### 备份节点1配置
```plaintext
vrrp_instance VI_1 {
    state BACKUP
    interface bond0
    virtual_router_id 51
    priority 90
    advert_int 1
    virtual_ipaddress {
        192.168.1.100
    }
}
```

##### 备份节点2配置
```plaintext
vrrp_instance VI_1 {
    state BACKUP
    interface bond0
    virtual_router_id 51
    priority 80
    advert_int 1
    virtual_ipaddress {
        192.168.1.100
    }
}
```

#### 如果 `virtual_router_id` 不一致会怎样？
如果 `virtual_router_id` 不一致，Keepalived 会将它们视为完全不同的虚拟路由器组，每个节点都会独立管理自己的 VIP，而不是协同工作。这会导致以下问题：
1. **VIP 在多个节点上同时绑定**：因为它们彼此之间没有同步机制。
2. **无法实现故障切换**：当主节点故障时，备份节点无法接管 VIP，因为它们认为自己属于不同的组。

#### 检查和修复
如果你怀疑 `virtual_router_id` 配置不一致，可以通过以下步骤检查和修复：
1. **检查配置文件**：
   - 登录到每个节点，查看 `/etc/keepalived/keepalived.conf` 文件，确认 `virtual_router_id` 是否一致。
   - 如果不一致，修改配置文件，使其保持一致。
2. **重新加载 Keepalived**：
   - 修改配置后，重新加载 Keepalived 服务：
     ```bash
     systemctl reload keepalived
     ```
3. **验证状态**：
   - 检查 Keepalived 的状态，确认 VIP 是否正确绑定到主节点：
     ```bash
     ip addr show bond0
     ```
   - 模拟故障切换，停止主节点的 Keepalived 服务，观察 VIP 是否漂移到备份节点。

#### 总结
`virtual_router_id` 是 Keepalived 中用于标识虚拟路由器的关键参数，必须在所有参与的节点上保持一致。如果不一致，会导致 VIP 管理混乱，无法实现高可用性。