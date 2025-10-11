# 记录CephFS运维问题

## 1、full ratio(s) out of order

~~~shell
# 设置 ​​“满”阈值​​。当任何 OSD 的使用率达到此比值时，​​集群将拒绝所有写入操作​​，以防止数据丢失。
ceph osd set-full-ratio 0.95
# 设置 ​​“近满”阈值​​。当任何 OSD 的使用率达到此比值时，集群会发出​​健康警告​​，提示空间即将耗尽。
ceph osd set-nearfull-ratio 0.85
# 设置 ​​“回填满”阈值​​。当任何 OSD 的使用率达到此比值时，集群会​​停止向该 OSD 回填或迁移数据​​，以避免加剧其空间压力。
ceph osd set-backfillfull-ratio 0.90
~~~

**阈值顺序必须正确**：这三个阈值的数值必须满足 **`backfillfull_ratio`< `nearfull_ratio`< `full_ratio`** 的关系。如果设置不当（例如 `nearfull_ratio`大于 `full_ratio`），Ceph 会报错 `full ratio(s) out of order`

**临时与永久设置**：使用 `ceph osd set-*-ratio`命令修改的是**运行时配置**，重启服务后可能会失效。为确保永久生效，建议在 Ceph 配置文件 `ceph.conf`中相应的配置项（如 `mon_osd_full_ratio`, `mon_osd_nearfull_ratio`, `mon_osd_backfillfull_ratio`）并重启服务

**调高阈值**（如从 0.95 调到 0.97）可以暂时缓解写入被阻塞的问题，但这是**治标不治本**的方法。本质上仍需通过**添加新 OSD** 或**清理数据**来扩容

## 2、记录异常