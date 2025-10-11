[toc]

## 复制表到指定空间

  ```she
  # 1. 置为不可用
  disable 'SV:MNR_CHUFA'
  # 2. 创建快照
  snapshot 'SV:MNR_CHUFA', 'MNR_CHUFA_NewSnapshot'
  # 3. 克隆快照
  clone_snapshot 'MNR_CHUFA_NewSnapshot', 'SV:MNR_CHUFA_NEW'
  # 4. 删除克隆快照
  delete_snapshot 'MNR_CHUFA_NewSnapshot'
  ```

## 获取table所有的rowkey

  ```
  count 'tablename', INTERVAL=>1
  ```

* **建表增加参数**
  参考：[建表参数](https://blog.csdn.net/yidu_fanchen/article/details/77188037)

        create 'test:default_TEST',{NAME => 'CF_L',COMPRESSION => 'SNAPPY' },{NAME => 'CF_V',COMPRESSION => 'SNAPPY' },{NAME => 'CF_R',COMPRESSION => 'SNAPPY' }, DURABILITY => 'SKIP_WAL'
  

## 通过HDFS文件恢复表数据

1. 先将完整的hbase数据文件复制到hbase数据目录下，注意工作空间需要对应  
   
```
# 文件上传HDFS,如果集群开启了kerberos认证，需要先进行认证
su hdfs
hadoop fs -put -d '表数据目录' 'hbase在hdfs上的数据目录'
# 将文件夹权限赋予 hbase 用户
hadoop fs -chown -R hbase:hbase 'hbase在hdfs上的数据目录'/'表数据目录'
# 
```

2. 重启HBASE, hbase会自动加载元数据
检查 '表数据目录' 是否被hbase正常加载

```
hbase shell
is_enable '表名'
```
如果正常进入下一步，不正常说明meta易损坏

3. 使用 hbck2 进行数据修复

[hbck2下载地址](https://dlcdn.apache.org/hbase/hbase-operator-tools-1.2.0/hbase-operator-tools-1.2.0-bin.tar.gz)

```
# 检查region
hbase hbck -j hbase-hbck2-1.2.0-SNAPSHOT.jar  reportMissingRegionsInMeta 

# 注册region
hbase hbck -j hbase-hbck2-1.2.0.jar  addFsRegionsMissingInMeta 工作空间名| 表名
hbase hbck -j hbase-hbck2-1.2.0.jar  assigns {上边命令输出的regionid}

# 输出 [-1,...] 后即为正常，重启hbase（未验证不重启是否正常）

# 检查表是否进入 transition 状态，进入则上述步骤成功
hbase shell

disbale 表名
enable 表名

# 验证数据是否正确加载，scan正常输出数据即成功恢复数据
scan 表名

```

* 查询指定RowKey数据
```bash
get 'DTSW_TEST:default_interference_wms_ltenr_ptosector_cellvalue_2024073143d','layerInfoRowKey'
```

hdfs dfs -setrep -R -w 3
