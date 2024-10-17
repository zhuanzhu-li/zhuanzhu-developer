[toc]

# 高性能MySQL

## 1. 架构

## 2. 基准测试

## 3. 服务器性能剖析

## 4. Schema 与数据类型优化

* 选择优化的数据类型

  * 更小通常更好
  * 简单就好
  * 尽量避免 NULL

* 支持类型

  * 整数

    TINYINT   SMALLINT  MEDIUMINT  INT  BIGINT

  * 实数 

    FLOAT DOUBLE DECIMAL

  * 字符串

    - VARCHAR 

      需要额外字节记录字符串长度，列的最大长度 <= 255 使用1个字节，否则使用2个字节；修改时需要更长时间

    - CHAR

  * BLOB TEXT

  * 使用枚举代替字符串

  * 日期和时间类型

    * DATATIME

      1001-9999 精度为秒 与时区无关

    * TIMESTAMP

      1970 - 2038 依赖于时区

  * 位数据

    * BIT
    * SET

## 5.创建高性能的索引

* 聚簇索引
* 覆盖索引
* 多列索引

## 6. 查询性能优化





