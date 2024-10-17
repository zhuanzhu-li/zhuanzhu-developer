nohup输入密码后继续在后台运行
一般在前台（即在当前终端，这里尤其指通过secureCRT或者putty连接远程终端）下运行短时间任务没有问题，但是如果运行长时间任务，比如传输大文件，比如编译大版本，就算你有功夫在前台等待但万一一不小心碰到了网线导致网络中断，就直接导致任务执行一半就挂掉，不得不重来的尴尬事件。

对此我们稍作优化便可以解决上述问题：

使用scp传输大文件
注意末尾不用加&


```
[mars@gms03 build]$ nohup scp MARS_2.2_1268.tar.gz 10.96.251.72:/data
nohup: appending output to `nohup.out'
Password:
```
```
输入密码后按：ctrl+z
```

1
```
[1]+  Stopped                 nohup scp MARS_2.2_1268.tar.gz 10.96.251.72:/data
```
然后紧接着输入：

1
```
[mars@gms03 build]$ bg
```
上述命令便又能在后台恢复运行了

1
```
[1]+ nohup scp MARS_2.2_1268.tar.gz 10.96.251.72:/data
```
