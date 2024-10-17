[toc]
# 基本安装
## 1. 安装准备

JDK 1.8以上
    
    链接：https://pan.baidu.com/s/1eWrqsI3yX1oH83I7H8BoyA?pwd=377a 
    提取码：377a 
    
GDAL 安装包

    官网下载：
    [gdal-3.4.1.tar.gz](https://github.com/OSGeo/gdal/releases/download/v3.4.1/gdal-3.4.1.tar.gz)
    
##  2. 配置环境


# linux安装

问题：

postgresql 支持
libpq

    ./configure  --with-pg --with-freexl --with-expat
    # ubantu
    apt-get install libpq-dev
    


# 完整gdal安装（包含绝大部分插件）

## 前置依赖包

* expat [http://sourceforge.net/projects/expat/files/expat/](http://sourceforge.net/projects/expat/files/expat/)

        mkdir /usr/lib64/expat201
        #指定安装目录：
        ./configure --prefix=/usr/local/expat201
        #生成安装文件：
        make
        #安装：
        make install
* curl [http://curl.haxx.se/download.html](http://curl.haxx.se/download.html)

        mkdir /usr/lib64/curl7240
        ./configure --prefix=/usr/local/curl7240
        make
        make install
* zlib [http://sourceforge.net/projects/libpng/files/zlib/](http://sourceforge.net/projects/libpng/files/zlib/)

        mkdir /usr/local/zlib123
        #64位系统下继续安装zlib会出现“could not read symbols: Bad value”错误，配置时采用64进行编译，如下：
        CFLAGS="-O3 -fPIC" ./configure --prefix=/usr/local/zlib123
        make
        make install

## 1.支持libkml

* libkml安装

        # 使用git 拉取源码
        git clone 'https://github.com/google/libkml.git'
        # gdal1.8以上需要libkml1.3.0以上，因此更改tag 
        git checkout -b libkml-1.3.0
        #进入源码目录，更新aclocal.m4
        aclocal
        #自动生成configure脚本：
        ./autogen.sh
        ./configure --prefix=/usr/local/libkml130 --with-expat-include-dir=/usr/local/expat201/include --with-expat-lib-dir=/usr/local/expat201/lib
        make && make install

        




# windows 安装

参考：[GDAL-JAVA环境安装](https://blog.csdn.net/qq_34281865/article/details/124933348)

## 遇到的问题：
geosmakevalidwithparams_r 相关的问题

注意环境变量配置