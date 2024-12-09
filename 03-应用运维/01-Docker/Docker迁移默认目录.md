# Docker Kubernetes迁移默认目录

## 停止服务

```shell
echo "停止之前 检查docker状态"
systemctl status docker.service | grep Active
echo "stop docker"
systemctl stop docker.service
systemctl stop docker.socket
echo "停止之前，检查kubelet状态"
systemctl status kubelet | grep Active
systemctl stop kubelet
echo "都停止完后，检查状态"
systemctl status docker.service | grep Active
systemctl status kubelet | grep Active
```

## 复制目录文件

```shell
mkdir -p /data1/docker
rsync -avz /var/lib/docker/* /data1/docker
vim /etc/docker/daemon.json
```

```json
"data-root":"/data1/docker",
```

```shell
systemctl start docker.socket
systemctl start docker.service
systemctl start kubelet
```
