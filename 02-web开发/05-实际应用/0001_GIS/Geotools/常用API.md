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