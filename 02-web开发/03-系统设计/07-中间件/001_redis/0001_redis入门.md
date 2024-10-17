[toc]

# 1.redis 简介

redis官方文档：[https://redis.io/docs/](https://redis.io/docs/)
参考blog
[一文搞懂redis](https://zhuanlan.zhihu.com/p/490268710)




# 2. Spring Boot接入redis


# 3.redis 事务

redis事务不支持原子性，即在运行过程中，单个提交错误，不会影响其他提交的执行，也不会进行回滚。

 <img src="../../99_source/img/redis-watchkey.jpg" width="500px">




# 4. redis持久化

## 4.1 RDB

​	RDB持久化产生一个压缩的二进制文件，可以通过这个文件进行redis数据还原操作。可以手动执行，也可以在 `redis.conf`中配置定时执行。

**工作原理**

1. Redis 调用forks。同时拥有父进程和子进程。
2. 子进程将数据集写入到一个临时 RDB 文件中。
3. 当子进程完成对新 RDB 文件的写入时，Redis 用新 RDB 文件替换原来的 RDB 文件，并删除旧的 RDB 文件。

**触发机制**

*主动触发*

bgsave

【注意】save会阻塞主线程，导致无法接受客户端请求



## 4.2 AOF

# 5. 发布订阅



# 6. 主从模式



# 7. 哨兵模式



# 8. 问题

## 8.1 缓存穿透

## 8.2 缓存击穿

## 8.3 缓存雪崩







​    
