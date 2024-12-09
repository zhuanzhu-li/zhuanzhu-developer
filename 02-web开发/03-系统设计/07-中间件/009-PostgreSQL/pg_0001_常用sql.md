* **sql 实现字段增加固定天数：**
  https://cloud.tencent.com/developer/ask/sof/591359
~~~sql
  update dgis_datamanager_model_life_time set data_cyle_time = data_create_time +  INTERVAL '7 days' 
  where data_mesuccess_time = '2022-04-06 14:14:57.101'
~~~

* **查询连接数 & 关闭连接方法**
~~~ sql
  -- 查询连接
  select * from pg_stat_activity;
  
  -- 关闭连接
~~~

* **多行转一行**
~~~ sql
select string_agg(column_name,split_char) from table
~~~

* **计算时间差**
~~~ sql
-- column_name 查询的列名 ，为timestamp格式时 转换为 天数
SELECT now( ) - ( column_name / 1000 / 60 / 60 / 24 || ' day' ) :: INTERVAL from table_name
~~~

* **字符串分割**
~~~ sql
--1.  SPLIT_PART
-- string : 待分割的字符串
-- delimiter：指定分割字符串
-- position：返回第几个字串，从1开始，该参数必须是正数。如果参数值大于分割后字符串的数量，函数返回空串。
select SPLIT_PART(string, delimiter, position)

-- 2. STRING_TO_ARRAY
-- string_to_array(string, delimiter [, null string])
-- 
-- string : 待分割的字符串
-- delimiter：指定分割字符串
-- null string : 设定空串的字符串
SELECT string_to_array('xx~^~yy~^~zz', '~^~', 'yy'); -- {xx,,zz}

-- 3. regexp_split_to_array
-- regexp_split_to_array ( string text, pattern text [, flags text ] ) → text[]
SELECT t as item FROM regexp_split_to_table('foo    bar,baz', E'[\\s,]+') AS t;

-- 4. regexp_split_to_array
-- 返回 {the,quick,brown,fox,jumps}
select regexp_split_to_array('the,quick,brown;fox;jumps', '[,;]') AS subelements

-- 5.regexp_matches
select regexp_matches('hello how are you', 'h[a-z]*', 'g')   as words_starting_with_h
~~~

* **insert select巧用**
~~~sql
-- 字段值可以是定值 eg: INSERT INTO user select '1' as id, 'zhangsan' as name from user_old;
INSERT INTO {表1} SELECT {字段1} as {表1字段1},  {字段2} as {表1字段2} FROM {表2};
~~~

* 锁表查询
~~~sql
-- 查询ACTIVITY的状态等信息
select T.PID, T.STATE, T.QUERY, T.WAIT_EVENT_TYPE, T.WAIT_EVENT,
T.QUERY_START
from PG_STAT_ACTIVITY T
where T.DATNAME = '数据库名';
-- 查询死锁的ACTIVITY
select T.PID, T.STATE, T.QUERY, T.WAIT_EVENT_TYPE, T.WAIT_EVENT,
T.QUERY_START
from PG_STAT_ACTIVITY T
where T.DATNAME = '数据库名'
and T.WAIT_EVENT_TYPE = 'Lock';
--  将第二条查询语句的pid字段的数字值记录下来，执行下面的查询语句可以解锁：
-- 通过pid解锁对应的ACTIVITY
select PG_CANCEL_BACKEND('6984');

~~~

* **小数保留指的位数**
~~~sql
select CAST(col_name as NUMERIC(10, 4)) from db_name
~~~

* pg 通过虚拟行号进行去重
~~~sql
select * from (
select col_1,col_2,..., row_number() over(partition by 字段1,字段2,... order by xxx) as row_num

) tmp where row_num = 1;
~~~

* pg 导入csv文件
~~~sql
COPY your_table_name FROM '/path/to/your/file.csv' DELIMITER ',' CSV HEADER;
~~~

* pg 相同数据库服务，跨库同步表数据
~~~bash
pg_dump -U username -d database_name -t table_name > output_file.sql
psql -U username -d target_database_name -f output_file.sql
~~~
  
* pg 字符串截断
~~~sql
SELECT SUBSTRING('string' FROM 1 FOR 6);
~~~

* pg 查询所有表名
~~~sql
SELECT table_name
FROM information_schema.tables
WHERE table_schema = 'public';
~~~