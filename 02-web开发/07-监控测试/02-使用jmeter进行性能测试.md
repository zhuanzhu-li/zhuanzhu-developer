# jmeter性能测试
## jmeter配置
* 安装
  直接官网下载即可：
    [下载地址](https://dlcdn.apache.org//jmeter/binaries/apache-jmeter-5.6.3.zip)
  解压后打开 `bin/jmeter.bat`文件即可
* **创建一个 `Thread Group`**

![jmeter进行性能测试配置-tg.jpg](../../resource/screenshot/jmeter3.jpg)
    `numbers of Threads` 代表同时执行的线程数量
    `ramp-up periods` 代表每间隔多久启动新的线程
    `loop count`  代表循环执行次数

* **在`Thread Group`创建一个如图的样本，一般是http请求**
![jmeter-创建一个样本用于测试.jpg](../../resource/screenshot/jmeter-2.jpg)

* **创建监听器**

可以根据需要创建多个监听器，也可以加一些统计的图表
![jmeter创建一个监听器.jpg](../../resource/screenshot/jmeter1.jpg)

* 运行测试
![jmeter运行测试.jpg](../../resource/screenshot/jmeter4.jpg)

## 常用指标
