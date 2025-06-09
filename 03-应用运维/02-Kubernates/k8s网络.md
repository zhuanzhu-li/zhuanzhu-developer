# k8s网络

## 集群环境

kubeadm 自动部署、keepalive+haproxy 做高可用、flannle 做网络插件

### 问题1

如果集群使用 keepalived + haproxy 做高可用，集群的vip为 10.1.1.200，集群三台服务器ip分别是 10.1.1.110、10.1.1.111、10.1.1.112，然后有个服务端口为 30000；当客户端请求 10.1.1.200:30000 时，在网络层面，是怎样调用到最终处理请求的pod的？



在使用 **Keepalived + HAProxy** 实现高可用的 Kubernetes 集群中，客户端请求通过虚拟 IP（VIP）访问服务时，整个网络层面的调用流程涉及多个组件的协同工作。以下是详细的步骤解析：

### 1. **客户端请求到达 VIP**
客户端通过虚拟 IP `10.1.1.200` 和端口 `30000` 发起请求：
```
10.1.1.200:30000
```

### 2. **Keepalived 的作用**
**Keepalived** 是一个用于实现高可用的软件，它通过虚拟路由冗余协议（VRRP）管理虚拟 IP（VIP）。在集群中，Keepalived 确保 VIP `10.1.1.200` 始终指向一个可用的节点。

- **工作原理**：
  - Keepalived 在三台服务器（`10.1.1.110`、`10.1.1.111`、`10.1.1.112`）上运行。
  - 它通过 VRRP 协议选举一个主节点（Master），VIP `10.1.1.200` 会被绑定到主节点的网络接口上。
  - 如果主节点故障，Keepalived 会自动将 VIP 切换到备用节点（Backup）。

### 3. **HAProxy 的作用**
当客户端请求到达 VIP `10.1.1.200` 时，请求实际上到达了当前持有 VIP 的节点（假设为 `10.1.1.110`）。在这个节点上，**HAProxy** 会接管请求。

- **工作原理**：
  - HAProxy 是一个高性能的负载均衡器，它监听 VIP 的端口 `30000`。
  - HAProxy 会根据配置的负载均衡策略（如轮询、最少连接等）将请求转发到后端的 Kubernetes 节点。
  - 在 Kubernetes 集群中，HAProxy 的后端通常是所有 Kubernetes 节点的 IP 地址（`10.1.1.110`、`10.1.1.111`、`10.1.1.112`）。

### 4. **请求到达 Kubernetes 节点**
HAProxy 将请求转发到某个 Kubernetes 节点（假设为 `10.1.1.111`）。此时，请求的路径类似于：
```
客户端 -> 10.1.1.200:30000 -> HAProxy（10.1.1.110）-> 10.1.1.111:30000
```

### 5. **Kubernetes 节点上的 iptables 规则**
当请求到达 Kubernetes 节点 `10.1.1.111` 时，节点上的 `iptables` 规则会接管流量。Kubernetes 在每个节点上设置了 `iptables` 规则，用于处理 NodePort 类型的服务。

- **工作原理**：
  - Kubernetes 为每个 NodePort 服务创建了一条 `iptables` 规则，将进入节点的流量从 NodePort 转发到服务的 ClusterIP 和端口。
  - 例如，如果服务的 NodePort 是 `30000`，ClusterIP 是 `10.96.0.1`，端口是 `80`，那么 `iptables` 规则会将流量从 `10.1.1.111:30000` 转发到 `10.96.0.1:80`。

### 6. **kube-proxy 的作用**
`iptables` 规则将流量转发到服务的 ClusterIP 后，**kube-proxy** 会接管流量的进一步处理。

- **工作原理**：
  - `kube-proxy` 是 Kubernetes 的网络代理组件，运行在每个节点上。
  - 它会监听 Kubernetes API Server，获取服务和 Pod 的信息，并根据这些信息动态更新本地的 `iptables` 规则。
  - 当流量到达服务的 ClusterIP 时，`kube-proxy` 会根据负载均衡策略选择一个后端 Pod，并将流量转发到该 Pod 的 IP 地址和端口。

### 7. **Flannel 的作用**
**Flannel** 是 Kubernetes 的 Pod 网络插件，负责在集群内构建一个虚拟的 Pod 网络。它确保所有 Pod 之间可以直接通过 IP 地址通信。

- **工作原理**：
  - Flannel 为每个 Pod 分配一个独立的 IP 地址，并通过虚拟网络接口（如 `flannel.1`）将 Pod 网络连接到宿主机网络。
  - 当流量需要从一个节点的 Pod 发送到另一个节点的 Pod 时，Flannel 会通过封装（如 VXLAN）将流量发送到目标节点，并在目标节点解封装后转发到目标 Pod。

### 8. **请求到达目标 Pod**
经过 `iptables`、`kube-proxy` 和 Flannel 的处理后，请求最终到达目标 Pod：
1. **客户端请求**：客户端通过 `10.1.1.200:30000` 发起请求。
2. **Keepalived 转发**：请求到达当前持有 VIP 的节点（假设为 `10.1.1.110`）。
3. **HAProxy 负载均衡**：HAProxy 将请求转发到某个 Kubernetes 节点（假设为 `10.1.1.111`）。
4. **iptables 转发**：节点上的 `iptables` 规则将流量从 `10.1.1.111:30000` 转发到服务的 ClusterIP 和端口（如 `10.96.0.1:80`）。
5. **kube-proxy 负载均衡**：`kube-proxy` 根据负载均衡策略选择一个后端 Pod（如 `10.244.1.2:9376`）。
6. **Flannel 虚拟网络**：如果目标 Pod 在另一个节点上，Flannel 会通过虚拟网络将流量封装并发送到目标节点。
7. **目标 Pod 处理**：目标 Pod 接收到请求后，处理并返回响应。

### 9. **返回路径**
返回路径的处理过程与请求路径类似：
1. **Pod 返回响应**：目标 Pod 处理请求后，将响应发送回宿主机。
2. **Flannel 处理**：如果目标 Pod 在另一个节点上，Flannel 会将响应封装并发送回原始节点。
3. **kube-proxy 处理**：`kube-proxy` 接收到响应后，将其转发回 HAProxy。
4. **HAProxy 返回响应**：HAProxy 将响应返回给客户端。
5. **客户端接收响应**：客户端收到响应，完成整个通信过程。

### 10. **总结**
在使用 **Keepalived + HAProxy** 实现高可用的 Kubernetes 集群中，客户端通过 VIP 访问服务的请求处理流程如下：
1. 客户端请求到达 VIP `10.1.1.200:30000`。
2. Keepalived 确保 VIP 始终指向一个可用的节点。
3. HAProxy 在当前持有 VIP 的节点上接管请求，并根据负载均衡策略将请求转发到某个 Kubernetes 节点。
4. Kubernetes 节点上的 `iptables` 规则将流量转发到服务的 ClusterIP 和端口。
5. `kube-proxy` 根据负载均衡策略选择后端 Pod。
6. Flannel 网络插件负责跨节点的 Pod 通信。
7. 请求最终到达目标 Pod，Pod 处理请求并返回响应。
8. 返回路径的处理过程与请求路径类似，最终客户端收到响应。

通过这种机制，Kubernetes 实现了服务的高可用性和负载均衡，同时确保 Pod 之间的通信高效且透明。