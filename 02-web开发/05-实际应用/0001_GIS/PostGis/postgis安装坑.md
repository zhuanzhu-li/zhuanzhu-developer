## libmpfr.so.4: 无法打开共享对象文件: 没有那个文件或目录


解决方式：
    
    # 安装 mpfr
    yum install mpfr
    # 链接
    ln -s /usr/lib/libmpfr.so.4.0.1 /usr/lib/libmpfr.sp.4
    # 配置 LD_LIBRARY_PATH
    vi ~/.bash_profile
    # 添加
    /usr/lib:/usr/lib64:/lib:/lib64:/usr/local/lib:/usr/local/lib64:/usr/lib:/usr/lib64:/lib:/lib64:/usr/local/lib:/usr/local/lib64: 
    source ~/.bash_profile
    # 最后重新执行 CREATE EXTENSION postgis;