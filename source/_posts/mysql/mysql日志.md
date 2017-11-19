---
title : mysql日志
date : 2017-11-10 09:00:00
author : imkindu
categories :
- mysql
- database
tags :
- mysql日志
---

　　日志文件对于一个服务器来说是非常重要的，它记录着服务器的运行信息，许多操作都会写日到日志文件，通过日志文件可以监视服务器的运行状态及查看服务器的性能，还能对服务器进行排错与故障处理，MySQl中有六种不同类型的日志。

<!--more-->

## 日志分类

　　mysql日子主要有以下几种类型：

> 1、查询日志 general_log
> 2、错误日志 log_error，log_warings
> 3、慢查询日志 log_show_queries
> 4、二进制日志 binary log
> 5、中继日志 relay_log
> 6、事务日志 transaction log




## 查询日志

　　说明：对除了慢查日志中记录的查询信息都将记录下来,这将对服务器主机产生大量的压力，所以对于繁忙的服务器应该关闭这个日志

记录查询语句，日志存储位置

- 文件：file
- 表：table（mysql.general_log）

``` sql
general_log=(on|off)                # 开启查询日志，默认是关闭的
general_log_file=HOSTNAME.log       # /mydata/data/mysql.log 指定查询日志的位置，默认在数据目录下
log_output={FILE|TABLE|NONE}        # 指定存放查询日志的位置，可以放在文件中，也可以放在数据库的表中，放在表中比放在文件中更容易查看
```


**a) 查询日志查询记录是否开启**

``` sql
MariaDB [(none)]> show global variables like 'general%';
+------------------+--------------+
| Variable_name    | Value        |
+------------------+--------------+
| general_log      | OFF          |
| general_log_file | centos20.log |
+------------------+--------------+
2 rows in set (0.00 sec)
```


**b) 开启日志查询记录**
``` sql
MariaDB [(none)]> set @@global.general_log=on;
```


## 慢查询日志


慢查询：运行时间超出指定时长的查询

> long_query_time

### 存储位置

>文件：FILE
表：TABLE，mysql.slow_log


### 相关配置选项

``` sql
log_slow_queries={ON|OFF}           # 是否记录慢查询日志
slow_query_log={ON|OFF}
slow_query_log_file=                # 慢查询日志文件名
log_output={FILE|TABLE|NONE}
log_slow_filter=                    # 慢查询过滤器
    admin,filesort,filesort_on_disk,full_join,full_scan,
    query_cache,query_cache_miss,tmp_table,tmp_table_on_disk
log_slow_rate_limit = 1             # 指明记录的速率（默认每秒记录一次）
log_slow_verbosity                  # 日志记录详细与否
```

### 查询慢查询时间定义

``` sql
MariaDB [(none)]> show global variables like 'long_query_time'\G
*************************** 1. row ***************************
Variable_name: long_query_time
        Value: 10.000000
1 row in set (0.00 sec)
```
### 设置慢查询时间

``` sql
# MYSQL的用户变量(@)和系统变量(@@)

MariaDB [(none)]> set @@GLOBAL.long_query_time=5;
Query OK, 0 rows affected (0.00 sec)

MariaDB [(none)]> show global variables like 'long_query_time';
+-----------------+----------+
| Variable_name   | Value    |
+-----------------+----------+
| long_query_time | 5.000000 |
+-----------------+----------+
1 row in set (0.00 sec)
```

        
### slow相关全局变量

``` sql
MariaDB [(none)]> show global variables like '%slow%'\G
*************************** 1. row ***************************
Variable_name: log_slow_filter              #
        Value: admin,filesort,filesort_on_disk,full_join,full_scan,
        query_cache,query_cache_miss,tmp_table,tmp_table_on_disk
*************************** 2. row ***************************
Variable_name: log_slow_queries
        Value: OFF
*************************** 3. row ***************************
Variable_name: log_slow_rate_limit
        Value: 1
*************************** 4. row ***************************
Variable_name: log_slow_verbosity
        Value: 
*************************** 5. row ***************************
Variable_name: slow_launch_time
        Value: 2
*************************** 6. row ***************************
Variable_name: slow_query_log               # 开启关闭慢查询日志
        Value: OFF
*************************** 7. row ***************************
Variable_name: slow_query_log_file          # 日志保存位置
        Value: centos20-slow.log
7 rows in set (0.00 sec)
```


### 配置文件永久配置
``` sql
[root@localhost my.cnf.d]# cat server.cnf

[mysqld]
# 域名反解析和INNODB单表记录
skip_name_resolve
innodb_file_per_table = ON
# 日志相关配置
slow_query_log = ON
log_output = TABLE
log_slow_queries = ON
long_query_time = 1
```

``` sql
MariaDB [mysql]> desc mysql.slow_log;
+----------------+------------------+------+-----+-------------------+-----------------------------+
| Field          | Type             | Null | Key | Default           | Extra                       |
+----------------+------------------+------+-----+-------------------+-----------------------------+
| start_time     | timestamp(6)     | NO   |     | CURRENT_TIMESTAMP | on update CURRENT_TIMESTAMP |
| user_host      | mediumtext       | NO   |     | NULL              |                             |
| query_time     | time(6)          | NO   |     | NULL              |                             |
| lock_time      | time(6)          | NO   |     | NULL              |                             |
| rows_sent      | int(11)          | NO   |     | NULL              |                             |
| rows_examined  | int(11)          | NO   |     | NULL              |                             |
| db             | varchar(512)     | NO   |     | NULL              |                             |
| last_insert_id | int(11)          | NO   |     | NULL              |                             |
| insert_id      | int(11)          | NO   |     | NULL              |                             |
| server_id      | int(10) unsigned | NO   |     | NULL              |                             |
| sql_text       | mediumtext       | NO   |     | NULL              |                             |
+----------------+------------------+------+-----+-------------------+-----------------------------+
11 rows in set (0.00 sec)
```


## 错误日志

记录如下几类信息：

> (1) mysqld启动和关闭过程中输出的信息； 
(2) mysqld运行中产生的错误信息； 
(3) event scheduler运行时产生的信息；
(4) 主从复制架构中，从服务器复制线程启动时产生的日志；


### 错误日志配置参数

``` sql
log_error=/var/log/mariadb/mariadb.log | OFF
log_warnings={ON|OFF}       # 是否记录警告信息至错误日志文件中
```

``` shell
[root@centos80 mariadb]# cat /var/log/mariadb/mariadb.log 
171113 20:10:03 mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
171113 20:10:03 [Note] /usr/libexec/mysqld (mysqld 5.5.52-MariaDB) starting as process 10200 ...
171113 20:10:03 InnoDB: The InnoDB memory heap is disabled
171113 20:10:03 InnoDB: Mutexes and rw_locks use GCC atomic builtins
171113 20:10:03 InnoDB: Compressed tables use zlib 1.2.7
171113 20:10:03 InnoDB: Using Linux native AIO
171113 20:10:03 InnoDB: Initializing buffer pool, size = 128.0M
171113 20:10:03 InnoDB: Completed initialization of buffer pool
InnoDB: The first specified data file ./ibdata1 did not exist:
InnoDB: a new database to be created!
171113 20:10:03  InnoDB: Setting file ./ibdata1 size to 10 MB
```


## 二进制日志

　　用于记录引起数据改变或存在引起数据改变的潜在可能性的语句（STATEMENT）或改变后的结果（ROW），也可能是二者混合；主要作用：“重放”

### 开启二进制日志

在/etc/my.cnf文件中【mysqld】下加上

``` shell
 server-id = 1 （在整个Mysql集群中保证唯一）
 log-bin = binlog 
 log-bin-index = binlog.index
```


### 配置文件永久开启二进制日志

``` shell
# 二进制日志
[mysqld]
server-id = 80
log-bin = binlog
log-bin-index = binlog.index
```

### 查看二进制日志
``` shell
# 在数据目录会出现binlog.00000x和binlog.index两个和二进制日志相关文件
[root@centos80 mysql]# ls
aria_log.00000001  binlog.000001  ibdata1      ib_logfile1  mysql       ON                  test
aria_log_control   binlog.index   ib_logfile0  imkindu      mysql.sock  performance_schema
# 日志文件类型
[root@centos80 mysql]# file binlog.000001 
binlog.000001: MySQL replication log
```


### 二进制日志查看工具

**查看当前系统管理的二进制日志文件列表**


SHOW MASTER|BINARY LOGS;

``` sql
MariaDB [(none)]> SHOW BINARY LOGS;
+---------------+-----------+
| Log_name      | File_size |
+---------------+-----------+
| binlog.000001 |       245 |
+---------------+-----------+
1 row in set (0.00 sec)
```

正在使用的是编号最大的那个文件，Mysql重启，编号会自动滚动加1.


**查看正在使用的二进制文件**

SHOW MASTER STATUS；

``` sql
MariaDB [(none)]> show master status;
+---------------+----------+--------------+------------------+
| File          | Position | Binlog_Do_DB | Binlog_Ignore_DB |
+---------------+----------+--------------+------------------+
| binlog.000001 |      245 |              |                  |
+---------------+----------+--------------+------------------+
1 row in set (0.00 sec)
```

**查询日志详细信息**

SHOW BINLOG EVENTS	 [IN 'log_name'] [FROM pos] [LIMIT [offset,] row_count]

``` sql
MariaDB [(none)]> show BINLOG EVENTS IN 'binlog.000001';
+---------------+-----+-------------+-----------+-------------+---------------------------------------------------------------------------------------------------------+
| Log_name      | Pos | Event_type  | Server_id | End_log_pos | Info                                                                                                    |
+---------------+-----+-------------+-----------+-------------+---------------------------------------------------------------------------------------------------------+
| binlog.000001 |   4 | Format_desc |        81 |         245 | Server ver: 5.5.52-MariaDB, Binlog ver: 4                                                               |
| binlog.000001 | 245 | Query       |        81 |         423 | GRANT REPLICATION CLIENT,REPLICATION SLAVE ON *.* TO 'repluser'@'192.168.56.%' IDENTIFIED BY 'replpass' |
| binlog.000001 | 423 | Query       |        81 |         498 | flush privileges                                                                                        |
| binlog.000001 | 498 | Query       |        81 |         562 | BEGIN                                                                                                   |
| binlog.000001 | 562 | Query       |        81 |         665 | insert into imkindu.user values (4,'dongge')                                                            |
| binlog.000001 | 665 | Xid         |        81 |         692 | COMMIT /* xid=36 */                                                                                     |
| binlog.000001 | 692 | Query       |        81 |         756 | BEGIN                                                                                                   |
| binlog.000001 | 756 | Query       |        81 |         860 | insert into imkindu.user values (5,'aoteman')                                                           |
| binlog.000001 | 860 | Xid         |        81 |         887 | COMMIT /* xid=47 */                                                                                     |
+---------------+-----+-------------+-----------+-------------+---------------------------------------------------------------------------------------------------------+
9 rows in set (0.00 sec)

```

`Log_name`： 日志文件名
`Pos position` ： 当前事件在日志文件第几位（二进制文件）
`Event_type` ： 事件类型
`Server_id` ： 服务器编号
`End_log_pos` ： 下一个事件开始为止


### mysqlbinlog

shell> mysqlbinlog [options] log_file ...

用于查看二进制日志内容

``` shell   
    YYYY-MM-DD hh:mm:ss

    --start-datetime=
    --stop-datetime=
    
    -j, --start-position=#
    --stop-position=#
    
    --user, --host, --password
```

### 二进制格式

binlog_format={STATEMENT|ROW|MIXED}

> STATEMENT：语句；记录执行的sql语句
ROW：行；记录执行sql语句前后修改行的数据（修改前和修改后），而不是记录sql语句
MIXED：混编；由数据库自行决定


### 服务器变量


sql_log_bin = ON | OFF ： 是否记录二进制日志
log_bin=/PATH/TO/BIN_LOG_FILE ：记录的文件位置，通常为OFF
session.sql_log_bin={ON|OFF} ： 控制某会话中的“写”操作语句是否会被记录于日志文件中；
max_binlog_size=1073741824 	: 字节 1G
sync_binlog={1|0} 			: 是否启动二进制日志同步功能（每句sql）
    

### 二进制日志事件格式

```text
# at 553
#160831  9:56:08 server id 1  end_log_pos 624   Query   thread_id=2     exec_time=0     error_code=0
SET TIMESTAMP=1472608568/*!*/;
BEGIN (例如：CREATE TABLE tb1 (id int, name char(30))) 
/*!*/;

事件的起始位置：# at 553
事件发生的日期时间：#160831  9:56:08
事件发生的服务器id：server id 1
事件的结束位置：end_log_pos 624
事件的类型：Query
事件发生时所在服务器执行此事件的线程的ID： thread_id=2 
语句的时间戳与将其写入二进制日志文件中的时间差：exec_time=0
错误代码：error_code=0
设定事件发生时的时间戳：SET TIMESTAMP=1472608568/*!*/;
事件内容：BEGIN
```



## 中继日志

relay_log 从服务器上记录下来从主服务器的二进制日志文件同步过来的事件；
			
			
## 事务日志

事务型存储引擎innodb用于保证事务特性的日志文件：

redo log 
undo log 