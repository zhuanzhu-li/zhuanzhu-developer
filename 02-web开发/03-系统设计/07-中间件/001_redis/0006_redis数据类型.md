# Redis数据类型

Redis作为高性能的键值存储系统，其丰富的数据类型使其能够适用于各种业务场景。下面我将详细介绍Redis支持的各种数据类型及其适用范围、常用命令和底层实现方式。

## 一、基础数据类型

### 1. String（字符串）

**适用范围**：

- 缓存简单的键值对数据（如用户会话、商品信息）
- 计数器（如文章阅读量、点赞数）
- 分布式锁的实现
- 存储JSON格式的数据

**常用命令**：

- `SET key value [EX seconds]`：设置键值，可设置过期时间
- `GET key`：获取键值
- `INCR key`/`DECR key`：原子递增/递减计数器
- `MSET key1 value1 [key2 value2...]`：批量设置值
- `GETSET key new_value`：设置新值并返回旧值
- `APPEND key value`：追加字符串
- `STRLEN key`：获取字符串长度

**实现方式**：

- 使用SDS（Simple Dynamic String）结构存储字符串
- 数字类型会优化为整数操作（如INCR、DECR）
- 新版本中，小于44字节的字符串使用embstr（压缩字符串），否则使用raw格式

### 2. Hash（哈希表）

**适用范围**：

- 存储对象（如用户信息、商品详情）
- 购物车实现
- 需要对部分字段进行更新的场景

**常用命令**：

- `HSET key field value`：设置单个字段
- `HGET key field`：获取字段值
- `HMSET key field1 value1 [field2 value2...]`：批量设置字段（已弃用，建议用HSET）
- `HGETALL key`：获取所有字段（慎用大数据量）
- `HSCAN key cursor [MATCH pattern][COUNT count]`：安全遍历大哈希对象
- `HINCRBY key field increment`：增加字段的整数值

**实现方式**：

- 使用哈希表实现
- 当字段较少（小于512个）且单个元素小于64字节时，使用压缩列表（ziplist），否则使用哈希表

### 3. List（列表）

**适用范围**：

- 消息队列（生产者消费者模式）
- 最新消息排行（如最近的10条动态）
- 待办事项列表
- 历史记录（如用户最近浏览商品）

**常用命令**：

- `LPUSH/RPUSH key value`：左/右插入元素
- `LPOP/RPOP key`：左/右弹出元素
- `BLPOP/BRPOP key timeout`：阻塞式弹出（用于实时消息系统）
- `LRANGE key start stop`：范围查询（用于分页查询）
- `LTRIM key start stop`：修剪列表（维护固定长度队列）
- `LLEN key`：获取列表长度

**实现方式**：

- 使用双向链表或压缩列表实现
- 小列表（元素个数小于512且单个元素小于64字节）使用压缩列表，大列表使用链表

### 4. Set（集合）

**适用范围**：

- 去重（如记录唯一用户ID）
- 随机抽取元素（如抽奖活动）
- 共同关注、共同好友等社交关系
- 标签系统

**常用命令**：

- `SADD key member`：添加元素
- `SMEMBERS key`：获取所有成员
- `SISMEMBER key member`：检查成员是否存在
- `SUNION key1 key2`：并集运算
- `SINTER key1 key2`：交集运算
- `SDIFF key1 key2`：差集运算
- `SPOP key`：随机移除并返回一个元素

**实现方式**：

- 使用哈希表实现
- 当存储的数据是整数且不超过512个时，使用有序数组存储，否则使用哈希表

### 5. Sorted Set（有序集合）

**适用范围**：

- 排行榜（如游戏分数排名）
- 带权重的任务队列
- 电话和姓名排序
- 地理空间索引（基于分数存储位置信息）

**常用命令**：

- `ZADD key score member`：添加元素
- `ZRANGE key start stop [WITHSCORES]`：按分数升序获取范围
- `ZREVRANGE key start stop [WITHSCORES]`：按分数降序获取范围
- `ZSCORE key member`：获取成员的分数
- `ZREVRANK key member`：获取成员的降序排名
- `ZINCRBY key increment member`：增加成员的分数

**实现方式**：

- 使用跳跃表（skiplist）和字典实现
- 跳跃表用于排序，字典用于快速查找
- 当元素个数小于512且单个元素小于64字节时，使用压缩列表，否则使用跳表

## 二、高级数据类型

### 6. Bitmaps（位图）

**适用范围**：

- 用户签到记录
- 判断用户登录状态
- 统计活跃用户数
- 连续签到用户总数统计

**常用命令**：

- `SETBIT key offset value`：设置位的值（0或1）
- `GETBIT key offset`：获取位的值
- `BITCOUNT key [start end]`：统计值为1的位数
- `BITOP operation destkey key [key...]`：位运算（AND/OR/XOR/NOT）

**实现方式**：

- 基于String类型实现，使用位操作
- 通过最小的单位bit来进行0或者1的设置

### 7. HyperLogLog

**适用范围**：

- 统计大规模数据集的唯一值数量
- 百万级网页UV计数
- 海量数据基数统计

**常用命令**：

- `PFADD key element [element...]`：添加元素
- `PFCOUNT key [key...]`：估算基数
- `PFMERGE destkey sourcekey [sourcekey...]`：合并多个HyperLogLog

**实现方式**：

- 使用概率算法实现
- 占用内存小但有一定误差（标准误差0.81%）

### 8. GEO（地理空间）

**适用范围**：

- 存储地理位置信息
- 附近的人或地点查询
- 滴滴叫车等位置服务

**常用命令**：

- `GEOADD key longitude latitude member`：添加地理位置
- `GEODIST key member1 member2 [unit]`：计算两地距离
- `GEORADIUS key longitude latitude radius unit`：查找指定半径内的位置

**实现方式**：

- 使用GeoHash编码将经纬度转换为字符串
- 存储在有序集合中，分数表示GeoHash值

### 9. Stream

**适用范围**：

- 消息队列
- 事件溯源
- 相比于基于List类型实现的消息队列，支持消费组和自动生成全局唯一ID

**常用命令**：

- `XADD key ID field value [field value...]`：添加消息
- `XREAD [COUNT count] [BLOCK milliseconds] STREAMS key [key...] ID [ID...]`：读取消息
- `XGROUP CREATE key groupname ID`：创建消费组

**实现方式**：

- Redis 5.0引入的新数据类型
- 基于基数树（radix tree）和链表实现

## 三、Redis数据类型选择建议

1. **简单键值存储**：使用String类型
2. **对象存储**：使用Hash类型
3. **队列/栈实现**：使用List类型
4. **唯一性数据**：使用Set类型
5. **需要排序的场景**：使用Sorted Set类型
6. **二值状态统计**：使用Bitmaps
7. **基数估算**：使用HyperLogLog
8. **地理位置**：使用GEO类型
9. **高级消息队列**：使用Stream类型

Redis所有操作都是原子性的，这得益于其单线程执行模型。在选择数据类型时，除了考虑适用场景外，还应考虑内存使用效率和操作性能，特别是对于大数据量的情况。