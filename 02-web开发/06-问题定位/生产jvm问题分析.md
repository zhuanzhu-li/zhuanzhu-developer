* 在docker中使用jattach获取堆栈信息
``` shell
# 安装 jattach

apk add --allow-untrusted --repositories-file=jattach-1.5-r0.apk
apk add jattach-1.5-r0.apk


# dump heap
jattach 1 dumpheap /opt/dumpheap.hprof
jattach 1 threaddump > /opt/app/threaddump.hprof

```

* 使用jdk自带的工具

  **`jstat`（实时监控 JVM 内存与 GC）**

  ~~~sh
  jstat -gcutil <pid> 1000 5  # 每隔1秒采样1次，共5次
  ~~~

  **`jmap`（堆内存分析）**

  ~~~sh
  jmap -heap <pid>      # 查看堆内存配置与使用情况
  jmap -histo:live <pid> # 统计堆中存活对象分布（谨慎使用，触发 Full GC）
  jmap -histo:live 1 | head 10 # 查看对象前十
  ~~~

  **`jcmd`（多功能诊断）**

  ```
  jcmd <pid> GC.heap_info       # 获取堆内存摘要
  jcmd <pid> VM.native_memory   # 查看 Native 内存使用
  ```