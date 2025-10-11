[toc]


##  Envelop 坐标系转换

~~~ java
Envelope sourceBBox = JTS.transform(originalEnvelop,CRS.findMathTransform(sourceCrs,targetCrs));
~~~


## 合并Filter 

~~~ java

// 其他的连接方式参考 FilterFactory 的API
FilterFactory f = new FilterFactoryImpl();
f.and(f1,f2);
~~~

## geotools mavne仓库地址

https://repo.osgeo.org/repository/release/

~~~xml
<repository>
    <id>osgeo</id>
    <name>OSGeo Release Repository</name>
    <url>https://repo.osgeo.org/repository/release/</url>
    <snapshots><enabled>false</enabled></snapshots>
    <releases><enabled>true</enabled></releases>
</repository>


<!-- osgeo（伪）镜像 -->
<mirror>
    <id>mirror-osgeo</id>
    <mirrorOf>osgeo</mirrorOf>
    <name>Mirror Osgeo</name>
    <url>https://repo.osgeo.org/repository/release/</url>
</mirror>
~~~

