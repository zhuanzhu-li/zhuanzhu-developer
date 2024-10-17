## 简介



## 简单使用

使用Redis为用例

参考：http://www.geomesa.org/documentation/stable/user/redis/usage.html



​			

## 单机版安装

~~~shell

## GeoMesa安装过程
wget "https://github.com/locationtech/geomesa/releases/download/geomesa-${TAG}/geomesa-redis_3.3.3-bin.tar.gz"
## 本人是直接下载，然后上传至服务器的。github,geomesa官网有下载链接

## 解压到指定目录
tar -zxvf geomesa-redis_3.3.3-bin.tar.gz

## 验证是否正常
cd /*/geomesa-redis_2.12-3.3.0/bin
./geomesa-redis


## 添加 GEOMESA_REDIS 到环境变量
vim /etc/profile

# 增加以下内容
# GEOMESA_REDIS_HOME=/opt/software/geomesa-redis_2.12-3.3.0
# PATH=$PATH:$GEOMESA_REDIS_HOME/bin
# CLASSPATH=$CLASSPATH:$GEOMESA_REDIS_HOME/lib
# export GEOMESA_REDIS_HOME PATH CLASSPATH

source /etc/profile

## 测试
geomesa-redis

#
#INFO  Usage: geomesa-redis [command] [command options]
#  Commands:
#    classpath            Display the GeoMesa classpath
#    configure            Configure the local environment for GeoMesa
#    convert              Convert files using GeoMesa's internal converter framework
#    create-schema        Create a GeoMesa feature type
#    delete-catalog       Delete a GeoMesa catalog completely (and all features in it)
#    delete-features      Delete features from a GeoMesa schema
#    describe-schema      Describe the attributes of a given GeoMesa feature type
#    env                  Examine the current GeoMesa environment
#    explain              Explain how a GeoMesa query will be executed
#    export               Export features from a GeoMesa data store
#    gen-avro-schema      Generate an Avro schema from a SimpleFeatureType
#    get-sft-config       Get the SimpleFeatureType definition of a schema
#    get-type-names       List the feature types for a given catalog
#    help                 Show help
#    ingest               Ingest/convert various file formats into GeoMesa
#    manage-partitions    Manage partitioned schemas
#    ng                   Manage the Nailgun server used for executing commands
#    remove-schema        Remove a schema and all associated features
#    scala-console        Run a Scala REPL with the GeoMesa classpath and configuration loaded
#    stats-analyze        Analyze statistics on a GeoMesa feature type
#    stats-bounds         View or calculate bounds on attributes in a GeoMesa feature type
#    stats-count          Estimate or calculate feature counts in a GeoMesa feature type
#    stats-histogram      View or calculate counts of attribute in a GeoMesa feature type, grouped by sorted values
#    stats-top-k          Enumerate the most frequent values in a GeoMesa feature type
#    update-schema        Update a GeoMesa feature type
#    version              Display the GeoMesa version installed locally

~~~



