* 在docker中使用jattach获取堆栈信息
``` shell
# 安装 jattach

apk add --allow-untrusted --repositories-file=jattach-1.5-r0.apk
apk add jattach-1.5-r0.apk


# dump heap
jattach 1 dumpheap /opt/app/dumpheap.hprof
jattach 1 threaddump > /opt/app/threaddump.hprof

```