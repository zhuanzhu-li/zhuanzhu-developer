
# 单机模式
# 集群模式

* **建表**

``` sql
-- 创建本地表
----------------------------------
CREATE TABLE {数据库.表名称A} on cluster {集群名称} (`p_provincecode` Int32,
`p_date` String,
`p_hour` Int32,
...
`enodeb_type` String) 
 ENGINE = ReplicatedMergeTree('/clickhouse/tables/{table}/{shard}','{replica}')
 PARTITION BY ({字段})
 ORDER BY({字段})
 SETTINGS index_granularity = 8192,
 storage_policy = 'dgis_cluster_volumes_policy'


-- 创建分布式表 ，基于上表 {数据库.表名称A} 创建对应的 分布式表
------------------------------------
CREATE TABLE {数据库.表名称B} on cluster {集群名称} (`p_provincecode` Int32,
`p_date` String,
`p_hour` Int32,
.......
`enodeb_type` String) 
 ENGINE = Distributed({集群名称},{数据库},{表名称A},rand())
```

* 示例

```sql
-- 创建本地表
CREATE TABLE _4g_location on cluster olatv_gis (`datetime` Date),
`enodebid` String,
`cellid` String,
`imsi` String,
`ulsinr` String,
`phr` String,
`aoa` String,
`recvbsr` String,
`puschprbnum` String,
`pdschprbnum` String,
`rsrp` String,
`dlearfcn` String,
`pci` String,
`rsrq` String,
`ncellcount` String,
`neighcellearfcn` String,
`neighpci` String,
`neighcellrsrp` String,
`neighcellrsrq` String,
`ta` String,
`cqi` String,
`mmecode` String,
`mmes1apid` String,
`mmegroupid` String,
`plmn` String,
`mdn` String,
`mrlon` String,
`mrlat` String,
`neighcellenbid` String,
`neighcellid` String,
`locationtype` String,
`buildingid` String,
`buildingfloor` String,
`positionmark` String,
`imei` String,
`earth_id` Int32,
`x_off_set` Int32,
`y_off_set` Int32) 
 ENGINE = ReplicatedMergeTree('/clickhouse/tables/{table}/{shard}','{replica}')
 PARTITION BY (datetime)
 ORDER BY(datetime)
 SETTINGS index_granularity = 8192,
 storage_policy = 'dgis_cluster_volumes_policy'


-- 创建分布式表
CREATE TABLE _4g_location_cluster on cluster olatv_gis (`datetime` Date,
`enodebid` String,
`cellid` String,
`imsi` String,
`ulsinr` String,
`phr` String,
`aoa` String,
`recvbsr` String,
`puschprbnum` String,
`pdschprbnum` String,
`rsrp` String,
`dlearfcn` String,
`pci` String,
`rsrq` String,
`ncellcount` String,
`neighcellearfcn` String,
`neighpci` String,
`neighcellrsrp` String,
`neighcellrsrq` String,
`ta` String,
`cqi` String,
`mmecode` String,
`mmes1apid` String,
`mmegroupid` String,
`plmn` String,
`mdn` String,
`mrlon` String,
`mrlat` String,
`neighcellenbid` String,
`neighcellid` String,
`locationtype` String,
`buildingid` String,
`buildingfloor` String,
`positionmark` String,
`imei` String,
`earth_id` Int32,
`x_off_set` Int32,
`y_off_set` Int32) 
 ENGINE = Distributed(olatv_gis,default,_4g_location,rand())

```