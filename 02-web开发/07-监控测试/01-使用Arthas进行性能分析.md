# 使用Arthas进行性能分析
适用场景：java 8+
## 准备工作
* 启动参数中增加如下参数
     `-XX:+StartAttachListener`
* 如果Java使用的jre部署的，则需要 tools.jar 作为 外部依赖
* 将 tools.jar 放到 $JAVA_HOME/lib 目录下面
~~~bash
cp tools.jar /usr/lib/jvm/java-1.8-openjdk/jre/lib/
# 解压相关包
mkdir -p /root/.arthas/lib/4.0.5/arthas
unzip arthas-packaging-4.0.5-bin.zip -d /root/.arthas/lib/4.0.5/arthas
# 使用java启动arthas , 1 是监控的java程序进程号
java -jar arthas-boot.jar 1
~~~

## 常用分析命令
### 分析一段时间内的代码执行情况
~~~bash
# 300s
profiler start --duration 3000
~~~
![arthas分析代码cpu占有率.png](..\..\resource\screenshot\arthas分析代码cpu占有率.png)
### 将输出的html打开
呈现如下火焰图，具体分析即可
![arthas分析生成的火焰图.png](..\..\resource\screenshot\arthas分析生成的火焰图.png)