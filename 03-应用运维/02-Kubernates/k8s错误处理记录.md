# k8s错误处理记录

** 优先检查是否是环境变量是否加载正确（目前都是环境变量）

** k8s查看pod日志报错 remote error : tls: internal error

~~~bash
for i in $(kubectl get csr | awk '/Pending/ {print $1}'); do kubectl certificate approve $i; done
~~~