# hadoop java api 坑

1. 同一进程中，多个用户Principal

官方jira
[jira](https://issues.apache.org/jira/browse/HADOOP-16122?jql=project%20%3D%20HADOOP%20AND%20resolution%20%3D%20Unresolved%20AND%20text%20~%20%22multiple%20user%20principals%20in%20a%20process%22%20ORDER%20BY%20priority%20DESC%2C%20updated%20DESC)

参考：[一文搞懂hadoop中的用户](https://blog.csdn.net/hncscwc/article/details/124358282)

Re-login from keytab for multiple UGI will use the same and incorrect keytabPrincipal

结论： 尽量保证在同一个服务中使用同一个用户进行hadoop操作，可以增加hadoop配置代理用户，用于不同用户之间的切换（现有项目）



	2. FileSystem.Cache 的坑

线上出现 OOM 问题

解决方案

~~~
export HADOOP_CLIENT_OPTS="-Xmx4g"
~~~



