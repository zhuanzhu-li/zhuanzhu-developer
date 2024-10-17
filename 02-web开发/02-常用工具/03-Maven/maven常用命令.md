* 本地jar导入本地库

~~~shell
mvn install:install-file -Dfile=.\mapbox-vector-tile-3.1.0-jts-1.17-pbf-2.15.jar -DgroupId=com.wdtinc -DartifactId=mapbox-vector-tile -Dversion=3.1.0-jts-1.17-pbf-2.15 -Dpackaging=jar
~~~

* 本地jar推送到远程库

~~~shell
mvn deploy:deploy-file -Dfile=/path/to/jar/file -DgroupId=com.wdtinc -DartifactId=mapbox-vector-tile -Dversion=3.1.0-jts-1.17 -Dpackaging=jar -Durl=http://mvn.datasw.com/repository/maven-releases/ -DrepositoryId=my-repo
~~~

