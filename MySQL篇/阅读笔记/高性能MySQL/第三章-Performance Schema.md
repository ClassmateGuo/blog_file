# 第三章-Performance Schema

## 简介
MYSQL Performance schema(PFS)是mysql提供的强大的性能监控诊断工具，提供了一种能够在运行时检查server内部执行情况的特方法。PFS通过监视server内部已注册的事件来收集信息，一个事件理论上可以是server内部任何一个执行行为或资源占用，比如一个函数调用、一个系统调用wait、SQL查询中的解析或排序状态，或者是内存资源占用等。

PFS将采集到的性能数据存储在performance_schema存储引擎中，performance_schema存储引擎是一个内存表引擎，也就是所有收集的诊断信息都会保存在内存中。诊断信息的收集和存储都会带来一定的额外开销，为了尽可能小的影响业务，PFS的性能和内存管理也显得非常重要了。


Performance schema 介绍
了解Performance schema工作机制,需要先了解2个概念:
- 程序插桩(instrument),程序插桩在MySQL代码中插入探测代码,以获取需要的信息.
- 消费者表(consumer),指的是存储关于程序插桩代码信息的表.

![](https://img.syilun.top/Performance%20schema%E6%94%B6%E9%9B%86%E4%B8%8E%E8%81%9A%E5%90%88%E6%95%B0%E6%8D%AE.png)

## 插桩元件

performance_schema表中的setup_instruments表包含了所有支持的插桩的列表.
![setup_instruments](http://img.syilun.top/performance_schema@setup_instruments.png)

### 插桩名称都是用斜杠分隔的部件组成
例子:`statement/sql/select` -> select是sql子系统的一部分,属于statement类型.
插桩名称的最左边部分表示`插桩的类型`,名字字段中的其他部分从左至右依次表示从通用到特定的子系统

## 消费者表的组织
消费者表是插桩发送信息的目的地,测量结果存储在Performance schema数据库的多个表中;事实上,MySQL 8.0.25社区版的Performance schema中包含110个表.
![消费者表组织](http://img.syilun.top/MySQL-消费者表组织.png)

## 实例表
用于MySQL安装程序.
![](http://img.syilun.top/MySQL-Instance(实例表).png)

## 设置表
设置表用于performance_schema 的运行时设置.
![](http://img.syilun.top/MySQL-setup(设置表).png)

剩下的就是其他表.

## 资源消耗
Performance schema收集的数据保存在内存中.可以通过设置消费者表的最大大小来限制其使用的内存量.
每个插桩指令的调用都会再添加两个`宏调用`,以将数据存储在performance_schema中(这就意味着`插桩越多`,CPU的使用率就越高).对CPU使用率的实际影响取决于特定的插桩->例如,要扫描一个有一百行的InnoDB表,引擎需要设置和释放一百行锁.`如果对锁使用wait类型的插桩,CPU使用率可能会显著增加.`

## 局限性
设置和使用performance_schema,需要了解它的局限性.
- 它必须得到MySQL组件的支持.
- 它只在特定的插桩和用户启动后才收集数据.
- 它很难释放内存.

## sys Schema
MySQL 5.7版本引入了一个新的sys schema，sys是一个MySQL自带的系统库，在安装MySQL 5.7以后的版本，使用mysqld进行初始化时，会自动创建sys库。

sys库里面的表、视图、函数、存储过程可以使我们更方便、快捷的了解到MySQL的一些信息，比如哪些语句使用了临时表、哪个SQL没有使用索引、哪个schema中有冗余索引、查找使用全表扫描的SQL、查找用户占用的IO等，sys库里这些视图中的数据，大多是从performance_schema里面获得的。目标是把performance_schema的复杂度降低，让我们更快的了解DB的运行情况。

sys schema包含了一个sys_config表和100个视图:
其实我们经常用到的是sys schema下的视图，下面将主要介绍各个视图的作用，我们发现sys schema里的视图主要分为两类，一类是正常以字母开头的，共52个，一类是以 x$ 开头的，共48个。字母开头的视图显示的是格式化数据，更易读，而 x$ 开头的视图适合工具采集数据，显示的是原始未处理过的数据。

下面按类别来分析以字母开头的52个视图：
- host_summary：这个是服务器层面的，以IP分组，比如里面的视图host_summary_by_file_io；
- user_summary：这个是用户层级的，以用户分组，比如里面的视图user_summary_by_file_io；
- innodb：这个是InnoDB层面的，比如视图innodb_buffer_stats_by_schema；
- io：这个是I/O层的统计，比如视图io_global_by_file_by_bytes；
- memory：关于内存的使用情况，比如视图memory_by_host_by_current_bytes；
- schema：关于schema级别的统计信息，比如schema_table_lock_waits；
- session：关于会话级别的，这类视图少一些，只有session和session_ssl_status；
- statement：关于语句级别的，比如statements_with_errors_or_warnings；
- wait：关于等待的，比如视图waits_by_host_by_latency。

## 常用的查询语句
[参考链接](https://blog.csdn.net/l1028386804/article/details/89521908)
```
1,查看每个客户端IP过来的连接消耗了多少资源。
mysql> select * from host_summary;
 
2,查看某个数据文件上发生了多少IO请求。
mysql> select * from io_global_by_file_by_bytes;
 
3,查看每个用户消耗了多少资源。
mysql> select * from user_summary;
 
4,查看总共分配了多少内存。
mysql> select * from memory_global_total;
 
5,数据库连接来自哪里，以及这些连接对数据库的请求情况是怎样的？
查看当前连接情况。
mysql> select host, current_connections, statements from host_summary;
 
6,查看当前正在执行的SQL和执行show full processlist的效果相当。
mysql> select conn_id, user, current_statement, last_statement from session;
 
7,数据库中哪些SQL被频繁执行？
执行下面命令查询TOP 10最热SQL。
mysql> select db,exec_count,query from statement_analysis order by exec_count desc limit 10;
 
8,哪个文件产生了最多的IO，读多，还是写的多？
mysql> select * from io_global_by_file_by_bytes limit 10;
 
9,哪个表上的IO请求最多？
mysql> select * from io_global_by_file_by_bytes where file like ‘%ibd’ order by total desc limit 10;
 
10,哪个表被访问的最多？
先访问statement_analysis，根据热门SQL排序找到相应的数据表。
哪些语句延迟比较严重？
查看statement_analysis中avg_latency的最高的SQL。
mysql> select * from statement_analysis order by avg_latency desc limit 10;
 
11,或者查看statements_with_runtimes_in_95th_percentile视图。
mysql> select * from statements_with_runtimes_in_95th_percentile;
 
12,哪些SQL执行了全表扫描或执行了排序操作？
mysql> select * from statements_with_sorting;
 
 mysql> select * from statements_with_full_table_scans;
 
13,哪些SQL语句使用了临时表，又有哪些用到了磁盘临时表？
查看statement_analysis中哪个SQL的tmp_tables 、tmp_disk_tables值大于0即可。
mysql> select db, query, tmp_tables, tmp_disk_tables from statement_analysis where tmp_tables>0 or tmp_disk_tables >0 order by (tmp_tables+tmp_disk_tables) desc limit 20;
 
14,也可以查看statements_with_temp_tables视图。
mysql> select * from statements_with_temp_tables\G
 
15 哪个表占用了最多的buffer pool？
mysql> select * from innodb_buffer_stats_by_table order by allocated desc limit 10;
 
16,每个库（database）占用多少buffer pool？
mysql> select * from innodb_buffer_stats_by_schema order by allocated desc limit 10;
 
17,每个连接分配多少内存？
利用session表和memory_by_thread_by_current_bytes分配表进行关联查询。
mysql> select b.user, current_count_used, current_allocated, current_avg_alloc, current_max_alloc, total_allocated,current_statement from memory_by_thread_by_current_bytes a, session b where a.thread_id = b.thd_id;
 
18,MySQL自增长字段的最大值和当前已经使用到的值？
mysql> select * from schema_auto_increment_columns;
 
19,MySQL索引使用情况统计？
mysql> select * from schema_index_statistics;

20,MySQL有哪些冗余索引和无用索引？
mysql> select * from schema_redundant_indexes;
mysql> select * from schema_unused_indexes;
 
21,MySQL内部有多个线程在运行？
MySQL内部的线程类型及数量。
mysql> select user, count(*) from processlist group by user;
```

## 理解线程
MySQL服务端是多线程软件,它的每个组件都使用线程.每个线程至少有2个唯一标识符: 一个是操作系统线程ID,另一个是MySQL内部线程ID.操作系统线程ID可以通过相关工具查看,内部线程ID在大多数performance_schema表中以THREAD_ID命名(THREAD_ID 不等于 PROCESSKUST_ID).performance_schema表中的threads表包含了服务器中存在的所有线程.
⚠️:performance_schema 到处使用THREAD_ID,但是PROCESSKUST_ID只在threads表中可用.

## 启动或禁用Performance Schema
可以将变量performance_schema设置为ON或者OFF,这是一个读变量,要么在配置文件中更改,要么在MySQL服务器启动时通过命令参数更改.

## 启动或禁用插桩
三种方法启动或者禁用performance_schema插桩
- 使用setup_instruments表.
- 调用sys_schema中的ps_setup_enable_instrument存储过程.
- 使用Performance-schema-instrument启动参数.

## 启动或禁用消费者表
三种方法启动或禁用
- 使用Performance Schema中的setup_consumers表.
- 调用sys_schema中的ps_setup_enable_consumer或者ps_setup_disable_consuper存储过程.
- 使用performance-schema-consumer启动参数.

## 优化
- 优化特定对象的监控:
Performance Schema 可以针对特定对象类型,schema 和对象名称启用或禁用监控,在setyp_objects表中完成.
对象类型(OBJECT_TYPE)可以是EVENT,FUNCTION,PROCEDURE,TABLE,TRIGGER.还可以指定OBJECT_SCHEMA和OBJECT_NAME,并支持通配符.

- 优化线程的监控:
setup_threads 表包含可以监控的后台线程列表.
用户线程的设置存储在setup_actors表中.

## 调整Performance Schema的内存大小
Performance Schema 将数据存储在使用PERFORMANCE_SCHEMA 引擎的表中,该引擎将数据存储在内存中.
默认情况下,某些performance_schema 表会自动调整大小,其他的则有固定数量的行.可以通过更改启动变量来调整这些选项.
⚠️:具体命令百度,基本上也很难去做调整.

## 默认值
⚠️:MySQL 不同部分的默认值会随着版本的不同而改变,因此,建议阅读官方的用户参考手册而不是百度"专家"的文章帖子.
关于Performance Schema 的一些重要的默认参数:
5.7版本开始,Performance Schema 在默认情况下是启用的.大多数插桩默认是禁用的,只启用了全局,线程,语句和事务插桩.
8.0版本开始,默认情况下还启动了元数据锁和内存插件.

## 使用Performance Schema

### 检查SQL语句
要启用语句检测,需要启动statement 类型的插桩:
statement/sql -> SQL语句,如SELECT,或者CREATE TABLE.
statement/sp -> 存储过程控制.
statement/scheduler -> 事件调度器.
statement/com -> 命令,如quit,kill,DROP DATABASE,或者Binlog Dump(⚠️:有些命令是用户不可用的,只能由mysqld进程调用).
statement/abstract -> 包括四类命令:clone,Query,new packet 和relay_log.

### 常规SQL语句
Performance Schema 将语句指标存储在events_statements_current,events_statements_history和events_statements_history_long表中.
event_statement_history表中可以作为优化指标.

### 可用于查找需要优化的语句的视图:
statement_analysis 具有聚合统计信息的规范化语句视图,按每个规范化语句的总执行时间排序.
statement_with_errors_or_warnings 所有引起错误或者警告的规范化语句.
statement_with_full_table_scans 所有执行了全表扫描的规范化语句.
statement_with_runtimes_in_95th_percentile 所有平均执行时间在前95%的规范化语句.
statement_with_temp_tables 所有使用了临时表的规范化语句.

### 预处理语句
performance_schema_instances 表包含服务器中存在的所有预处理的语句.
⚠️:要启动预处理语句检测,需要启动以下插桩
- statement/sql/prepare_sql 文本协议中的PREPARE语句(通过MySQL CLI运行).
- statement/sql/execute_sql 文本协议中的EXECUTE语句(通过MySQL CLI运行).
- statement/com/Prepare 二进制协议中的PREPARE语句(通过MySQL C API访问).
- statement/com/EXECUTE 二进制协议中的EXECUTE语句(通过MySQL C API访问).

### 存储例程
启用存储例程检测,需要启动匹配`statement/sp/%`模式的插桩.
statement/sp/stmt插桩负责例程内部调用的语句,而其他插桩则负责跟踪事件.

### 语句剖析
events_stages_[current|history|history_long]表包含剖析信息.例如MySQL 在创建临时表,更新或者等待锁时花费了多少时间,要启用剖析,需要启动上述消费者表以及匹配`stage/%`模式的插桩.

### 检查读写性能
statement 类型的插桩可对于理解工作负载是受读还是受写限制非常有用,可以从统计各类型语句的`执行量`入手.(需要启动插桩)
```sql
SELECT EVENT_NAME,COUNT(EVENT_NAME)FROM events_statements_history_long GROUP BY EVENT_NAME;
```

想知道语句的延迟情况,可以按LOCK_TIME 列进行聚合.
```sql
SELECT EVENT_NAME,COUNT(EVENT_NAME),SUM(LOCK_TIME/1000000)AS latency_ms FROM events_statements_history_long GROUP BY EVENT_NAME ORDER BY latency_ms DESC;
```

想知道读取和写入的字节数和行数,采用全局变量`Handler_*`.

### 检查元数据锁
performance_schema 中的metadata_locks 表包含关于当前不同线程设置的锁的信息,以及处于等待状态的锁请求信息,可以通过这个表轻松缺人哪个线程阻塞了DDL请求,可以去选择终止或者等待它完成.
```sql
SELECT * processlist_id,object_type,lock_type,lock_status,source FROM metadata_locks JOIN threads ON (owner_thread_id=thread_id) WHERE object
_schema = 'employees' AND object_name = 'titles'\G
```

### 检查内存使用情况
performance_schema 启用`memory`类的插桩来启用内存检测.
performance_schema 将内存使用统计信息存储在摘要表中,摘要表的名称以`memory_summary_`前缀开头.
内存使用的聚合函数:
- global 按事件名全局聚合
- thread 按线程聚合:包括后台线程和用户线程
- account 按用户账号聚合
- host 按主机聚合
- user 按用户名聚合
![Img](http://img.syilun.top/High-performance-memory-*.png)

### 使用sys schema
视图`x$memory_by_thread_by_current_bytes`中的行是按照当前分配的内存降序排序的,可以很容易的找到哪个线程占用了大部分内存.
```sql
SELECT
	thread_id tid,
	USER,
	CURRENT_allocated ca,
	total_allocated 
FROM
	sys.`x$memory_by_thread_by_current_bytes` 
	LIMIT 9;
```
![Img](http://img.syilun.top/High-performance-%E6%9F%A5%E8%AF%A2%E5%8D%A0%E7%94%A8%E5%86%85%E5%AD%98%E7%9A%84%E7%BA%BF%E7%A8%8B.png)

### 查询占用最多内存的用户线程
```sql
SELECT * FROM sys.memory_by_thread_by_current_bytes ORDER BY current_allocated DESC;
```

![Img](http://img.syilun.top/High-performance-%E6%9F%A5%E8%AF%A2%E5%8D%A0%E7%94%A8%E5%86%85%E5%AD%98%E6%9C%80%E5%A4%9A%E7%9A%84%E7%94%A8%E6%88%B7%E7%BA%BF%E7%A8%8B.png)

### 检查变量
+ 服务器变量
    - 全局级. 
    - 会话级,针对当前所有打开的会话.
    - 源,所有当前变量值的来源.
+ 状态变量
    - 全局级.
    - 会话级,针对当前所有打开的会话.
    - 聚合维度.
    - 主机
    - 用户名
    - 账号
    - 线程
+ 用户变量
⚠️: 在5.7版本之前,服务器和状态变量是在information_schema 中配置的,这种配置有限制,只允许跟踪全局和当前会话,其他会会话中关于变量和状态的信息,以及关于用户变量的信息,都是不可访问的.在8.0版本中该information_schema 中的变量表在8.0版中都不再存在.

全局变量值被存储在`global_variables`中.
当前会话的会话变量被存储在`session_variables`表中.
其他的变量值的存储都在`*_variables`后缀的表中.
![Img](http://img.syilun.top/High-performance-*-variables.png)

### 检查最常见的错误
`events_errors_summary_global_by_error`表的结构,所有错误相关的聚合表都有类似的结构.
```sql
CREATE TABLE `events_errors_summary_global_by_error` (
  `ERROR_NUMBER` int DEFAULT NULL,
  `ERROR_NAME` varchar(64) DEFAULT NULL,
  `SQL_STATE` varchar(5) DEFAULT NULL,
  `SUM_ERROR_RAISED` bigint unsigned NOT NULL,
  `SUM_ERROR_HANDLED` bigint unsigned NOT NULL,
  `FIRST_SEEN` timestamp NULL DEFAULT '0000-00-00 00:00:00',
  `LAST_SEEN` timestamp NULL DEFAULT '0000-00-00 00:00:00',
  UNIQUE KEY `ERROR_NUMBER` (`ERROR_NUMBER`)
) ENGINE=PERFORMANCE_SCHEMA DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci
```

### 检查Performance Schema自身
默认情况下,如果Performance Schema被设为默认数据库,则不会对它的查询进行跟踪.如果需要查询对performance_schema的查询,那么需要先更新setup_actors表.

## 总结
启用Performance Schema ,按需动态的启动插桩和消费者表,可以通过它们提供的数据去解决可能存在的任何问题(例如:查询性能,锁定,磁盘I/O,错误等),充分利用sys Schema是解决常见问题的捷径,这样可以直接从MySQL中进行性能的测量.