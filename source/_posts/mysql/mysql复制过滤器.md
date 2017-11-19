---
title : mysql复制过滤器
date : 2017-11-15 21:00:00
author : imkindu
categories :
- mysql
- database
tags : 
- mysql复制过滤器
---

　　在mysql主从复制中，有一些主服务器中系统表和测试表，或者其他无需复制的库或者表是不需要被复制的，需要特别指定，以免被复制。

<!--more-->


　　复制过滤器表示可以指定只复制指定的表或者只复制指定的库，其他的表和库都不进行复制，两种方法可以实现：


## 主服务器实现

`主服务器实现`：主服务器指定只有指定的表或者库的修改信息才传递到从服务器的中继日志，但是主服务器出现问题之后会丢失未复制表和库的数据；

主服务器仅向二进制日志中记录有关特定数据库相关的写操作；这样从服务器就无法从主节点二进制日志获取日志进行同步了。

``` text
binlog_do_db= 逗号分隔的数据库名称列表
binlog_ignore_db=
```

**问题**：其它库的time-point recovery将无从实现

主服务器重要日志不写入二进制日志，虽然从服务器不能复制了，但是如果主服务器节点中断后，连主服务器也无法恢复上次备份之后的数据了。
所以不建议在主节点实现。
        


## 从服务器实现         

`从服务器实现`：所有二进制日志都传到从服务器的中继日志，但是sql_thread只执行指定表和库的修改语句，当主服务器出现故障，则不会丢失数据，所有建议采用从服务器的复制过滤；


``` text
SET GLOBAL replicate_do_db=库名           #指定只复制指定的库
SET GLOBAL Replicate_Ignore_DB=库名       #指定不复制指定的库 
SET GLOBAL Replicate_Do_Table=表名        #指定只复制指定表
SET GLOBAL Replicate_Ignore_Table=表名    #指定不复制指定表
# wild 通配符  
Replicate_Wild_Do_Table=                  #通配符匹配需要复制的表
Replicate_Wild_Ignore_Table=	
```


当指定了复制过滤器后，通过`SHOW SLAVE STATUS`;命令可以查看到指定的库和表。

建议此配置写入配置文件永久生效。

``` shell
STOP SLAVE;
#首先将从节点的功能关闭
SET @@global.replicate_ignore_table='mydb.tbl';
#我们将mydb数据库中的tbl表过滤掉，即从节点的mysql数据库不会包含此表
START SLAVE IO_THREAD,SQL_THREAD;
#开启从节点的IO和SQL线程
#配置完成，接下来测试
```


设置忽略库并查看

``` sql
# 设置不需要复制的库
MariaDB [imkindu]> set global replicate_ignore_db=imkindu;
Query OK, 0 rows affected (0.00 sec)

# 查看从库状态
MariaDB [imkindu]> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: 192.168.56.81
                  Master_User: repluser
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: binlog.000002
          Read_Master_Log_Pos: 629
               Relay_Log_File: relay-log.000007
                Relay_Log_Pos: 718
        Relay_Master_Log_File: binlog.000002
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes
              Replicate_Do_DB: 
          Replicate_Ignore_DB: imkindu              # 忽略复制的库
```