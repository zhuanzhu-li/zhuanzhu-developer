# 记录一次javaOOM问题排查过程


## 1. 问题描述
    生产环境出现微服务在长时间运行后，发生OOM问题，部分日志如下:
    Exception in thread "JobRpcExecute:RemoteFileStateJob-6-3" java.lang.OutOfMemoryError: GC overhead limit exceeded
	at org.apache.ibatis.reflection.MetaObject.setValue(MetaObject.java:127)
	at org.apache.ibatis.executor.resultset.DefaultResultSetHandler.applyAutomaticMappings(DefaultResultSetHandler.java:510)
	at org.apache.ibatis.executor.resultset.DefaultResultSetHandler.getRowValue(DefaultResultSetHandler.java:388)
	at org.apache.ibatis.executor.resultset.DefaultResultSetHandler.handleRowValuesForSimpleResultMap(DefaultResultSetHandler.java:342)
	at org.apache.ibatis.executor.resultset.DefaultResultSetHandler.handleRowValues(DefaultResultSetHandler.java:317)
	at org.apache.ibatis.executor.resultset.DefaultResultSetHandler.handleResultSet(DefaultResultSetHandler.java:290)
	at org.apache.ibatis.executor.resultset.DefaultResultSetHandler.handleResultSets(DefaultResultSetHandler.java:187)
	at org.apache.ibatis.executor.statement.PreparedStatementHandler.query(PreparedStatementHandler.java:64)
	at org.apache.ibatis.executor.statement.RoutingStatementHandler.query(RoutingStatementHandler.java:79)
	at org.apache.ibatis.executor.SimpleExecutor.doQuery(SimpleExecutor.java:63)
	at org.apache.ibatis.executor.BaseExecutor.queryFromDatabase(BaseExecutor.java:324)
	at org.apache.ibatis.executor.BaseExecutor.query(BaseExecutor.java:156)
	at org.apache.ibatis.executor.BaseExecutor.query(BaseExecutor.java:136)
	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:148)
	at org.apache.ibatis.session.defaults.DefaultSqlSession.selectList(DefaultSqlSession.java:141)
	at sun.reflect.GeneratedMethodAccessor130.invoke(Unknown Source)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.mybatis.spring.SqlSessionTemplate$SqlSessionInterceptor.invoke(SqlSessionTemplate.java:433)
	at com.sun.proxy.$Proxy61.selectList(Unknown Source)
	at org.mybatis.spring.SqlSessionTemplate.selectList(SqlSessionTemplate.java:230)
	at org.apache.ibatis.binding.MapperMethod.executeForMany(MapperMethod.java:137)
	at org.apache.ibatis.binding.MapperMethod.execute(MapperMethod.java:75)
	at org.apache.ibatis.binding.MapperProxy.invoke(MapperProxy.java:59)
	at com.sun.proxy.$Proxy75.queryFileTasksBySJID(Unknown Source)
	at com.datasw.server.service.impl.FileTaskServiceImpl.queryFileTasksBySJID(FileTaskServiceImpl.java:49)
	

## 2. 排查过程

* 日志分析

     发现每次出现OOM问题都是在执行一个文件列表读取时出现的，但在观察代码时，未发现会导致内存泄漏问题的代码（肉眼看0.0）
     
* 测试环境测试

    由于测试环境的数据量以及持续运行时间无法达到生产环境。也没有复现OOM问题
    
* 生产环境oom HEAP DUMP
        
        增加jvm参数：
           -XX:+HeapDumpOnOutOfMemoryError
           -XX:HeapDumpPath=/usr/local/oom

* j visualVM工具分析heapdump文件
    heapdump文件分析方法：
    
    

