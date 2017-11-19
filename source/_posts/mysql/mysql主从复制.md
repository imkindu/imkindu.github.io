---
title: mysql主从复制、主主复制、半同步复制
date: 2017-11-14 19:00:00
author: imkindu
categories: database
tags:
- mysql主从复制
- ProxySQL
---

　　主服务器（Master）负责网站NonQuery操作，从服务器负责Query操作，用户可以根据网站功能模特性块固定访问Slave服务器，或者自己写个池或队列，自由为请求分配从服务器连接。主从服务器利用MySQL的二进制日志文件，实现数据同步。二进制日志由主服务器产生，从服务器响应获取同步数据库。

<!--more-->

## 主从复制

　　Mysql主从复制主要依赖于主服务器的bin_log二进制日志和从服务器的relay_log中继日志

<div align="center">
![](/images/mysql/zz.png)
</div>

### 时间同步

　　做主从复制，首先得确保所有服务器时间节点一致

``` shell
[root@centos80 ~]# ntpdate 172.18.0.1
14 Nov 15:57:31 ntpdate[11811]: step time server 172.18.0.1 offset 34.712450 sec
[root@centos80 ~]# date
Tue Nov 14 15:57:39 CST 2017
```


### 开启主服务器bin_log

　　主从复制主要依靠的是从服务器去读主服务器的`bin-log`二进制日志，进行同步，所以必须开启主服务器的二进制日志。

``` shell
vim /etc/my.conf.d/server.conf
[mysqld]
# 二进制日志
server-id = 80
log-bin = binlog                    # 开启二进制日志
log-bin-index = binlog.index
```

``` sql
# 查看主服务器二进制日志状态
MariaDB [(none)]> show binary logs;
+---------------+-----------+
| Log_name      | File_size |
+---------------+-----------+
| binlog.000001 |       498 |
+---------------+-----------+
1 row in set (0.00 sec)
```


### 开启从服务器relay_log

　　在从服务器这边要开启`relay-log`中继日志，以记录同步到了什么位置。

``` shell
[mysqld]
skip_name_resolve = ON
innodb_file_per_table = ON
max_connections = 2000

relay_log = relay-log               # 开启中继日志
server-id = 82                      # 服务器编号
```



### 主节点授权账号

　　从服务器监听主服务器，需要主服务器授权`REPLICATION CLIENT`、`REPLICATION SLAVE`，以让从服务器监听和同步。

``` sql
MariaDB [(none)]> GRANT REPLICATION CLIENT,REPLICATION SLAVE ON *.* TO 'repluser'@'192.168.56.%' IDENTIFIED BY 'replpass';
```

### 从服务器MASTER TO主服务器

　　在从服务器指明哪一台是主服务器，从哪个位置开始同步等信息。

``` sql
MariaDB [(none)]> help change master to;
Name: 'CHANGE MASTER TO'
Description:
Syntax:
CHANGE MASTER TO option [, option] ...

option:         # 下面每一个都是单独的配置选项，用逗号分开
    MASTER_BIND = 'interface_name'
  | MASTER_HOST = 'host_name'
  | MASTER_USER = 'user_name'
  | MASTER_PASSWORD = 'password'
  | MASTER_PORT = port_num
  | MASTER_CONNECT_RETRY = interval
  | MASTER_HEARTBEAT_PERIOD = interval
  | MASTER_LOG_FILE = 'master_log_name'
  | MASTER_LOG_POS = master_log_pos
  | RELAY_LOG_FILE = 'relay_log_name'
  | RELAY_LOG_POS = relay_log_pos
  | MASTER_SSL = {0|1}
  | MASTER_SSL_CA = 'ca_file_name'
  | MASTER_SSL_CAPATH = 'ca_directory_name'
  | MASTER_SSL_CERT = 'cert_file_name'
  | MASTER_SSL_KEY = 'key_file_name'
  | MASTER_SSL_CIPHER = 'cipher_list'
  | MASTER_SSL_VERIFY_SERVER_CERT = {0|1}
  | IGNORE_SERVER_IDS = (server_id_list)

server_id_list:
    [server_id [, server_id] ... ]
```

　　配置从服务器监听主服务器，并指明从二进制什么位置开始同步。

``` sql
MariaDB [imkindu]> CHANGE MASTER TO MASTER_HOST='192.168.56.81', MASTER_USER='repluser',MASTER_PASSWORD='replpass',MASTER_LOG_FILE='binlog.000001',MASTER_LOG_POS=498;
```

　　查看从服务器状态

``` sql
MariaDB [imkindu]> show slave status\G
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 192.168.56.81
                  Master_User: repluser
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: binlog.000001
          Read_Master_Log_Pos: 498
               Relay_Log_File: relay-log.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: binlog.000001
             Slave_IO_Running: No                   # 复制要靠IO线程
            Slave_SQL_Running: No                   # 重复依靠SQL线程
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 498
              Relay_Log_Space: 245
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 0
1 row in set (0.00 sec)
```





### 启动从服务器线程-开始复制

``` sql
MariaDB [imkindu]> START SLAVE IO_THREAD,SQL_THREAD;
Query OK, 0 rows affected (0.00 sec)
```




## 主主复制




## 半同步化复制

　　由于Mysql的复制都是基于异步进行的，在特殊情况下不能保证数据的成功复制，因此在mysql5.5之后使用了来自google补丁，可以将Mysql的复制实现半同步模式。所以需要为主服务器加载对应的插件。在Mysql的安装目录下的lib/plugin/目录中具有对应的插件semisync_master.so，semisync_slave.so，其中semisync_master.so是主服务器上的插件，而semisync_slave.so则是从服务器上的插件。前提是做好主从复制。

``` shell
[root@centos80 ~]# ll /usr/lib64/mysql/plugin
total 5016
-rwxr-xr-x  1 root root   41336 Nov 15  2016 semisync_master.so         # 主节点插件
-rwxr-xr-x  1 root root   15984 Nov 15  2016 semisync_slave.so          # 从节点插件
```

### 在主节点安装 semisync_master

``` shell
INSTALL PLUGIN plugin_name SONAME 'shared_library_name'
```

- 1、在主服务器的mysql服务器上执行如下命令

``` shell
MariaDB [(none)]> install plugin rpl_semi_sync_master soname 'semisync_master.so'; # 安装模块
MariaDB [(none)]> set global rpl_semi_sync_master_enabled = 1;             #启用半同步复制主节点
MariaDB [(none)]> set global rpl_semi_sync_master_timeout = 1000;  # 超时时间
MariaDB [(none)]> show variables like '%semi%';   # 查看设置是否成功
```

- 2、查看semisync_master插件安装情况

``` sql
MariaDB [(none)]> show plugins;
+--------------------------------+----------+--------------------+--------------------+---------+
| Name                           | Status   | Type               | Library            | License |
+--------------------------------+----------+--------------------+--------------------+---------+
| binlog                         | ACTIVE   | STORAGE ENGINE     | NULL               | GPL     |
| mysql_native_password          | ACTIVE   | AUTHENTICATION     | NULL               | GPL     |
| mysql_old_password             | ACTIVE   | AUTHENTICATION     | NULL               | GPL     |
| CSV                            | ACTIVE   | STORAGE ENGINE     | NULL               | GPL     |
| MEMORY                         | ACTIVE   | STORAGE ENGINE     | NULL               | GPL     |
| MyISAM                         | ACTIVE   | STORAGE ENGINE     | NULL               | GPL     |
| MRG_MYISAM                     | ACTIVE   | STORAGE ENGINE     | NULL               | GPL     |
| BLACKHOLE                      | ACTIVE   | STORAGE ENGINE     | NULL               | GPL     |
| InnoDB                         | ACTIVE   | STORAGE ENGINE     | NULL               | GPL     |
| INNODB_RSEG                    | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_UNDO_LOGS               | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_TRX                     | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_LOCKS                   | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_LOCK_WAITS              | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_CMP                     | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_CMP_RESET               | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_CMPMEM                  | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_CMPMEM_RESET            | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_SYS_TABLES              | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_SYS_TABLESTATS          | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_SYS_INDEXES             | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_SYS_COLUMNS             | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_SYS_FIELDS              | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_SYS_FOREIGN             | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_SYS_FOREIGN_COLS        | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_SYS_STATS               | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_TABLE_STATS             | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_INDEX_STATS             | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_BUFFER_POOL_PAGES       | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_BUFFER_POOL_PAGES_INDEX | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_BUFFER_POOL_PAGES_BLOB  | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| XTRADB_ADMIN_COMMAND           | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_CHANGED_PAGES           | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_BUFFER_PAGE             | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_BUFFER_PAGE_LRU         | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| INNODB_BUFFER_POOL_STATS       | ACTIVE   | INFORMATION SCHEMA | NULL               | GPL     |
| FEDERATED                      | ACTIVE   | STORAGE ENGINE     | NULL               | GPL     |
| ARCHIVE                        | ACTIVE   | STORAGE ENGINE     | NULL               | GPL     |
| PERFORMANCE_SCHEMA             | ACTIVE   | STORAGE ENGINE     | NULL               | GPL     |
| Aria                           | ACTIVE   | STORAGE ENGINE     | NULL               | GPL     |
| FEEDBACK                       | DISABLED | INFORMATION SCHEMA | NULL               | GPL     |
| partition                      | ACTIVE   | STORAGE ENGINE     | NULL               | GPL     |
| rpl_semi_sync_master           | ACTIVE   | REPLICATION        | semisync_master.so | GPL     |
+--------------------------------+----------+--------------------+--------------------+---------+
43 rows in set (0.00 sec)
```

- 3、检查插件是否开启

``` sql
MariaDB [(none)]> show variables like '%semi%';
+------------------------------------+-------+
| Variable_name                      | Value |
+------------------------------------+-------+
| rpl_semi_sync_master_enabled       | OFF   | # 未启用
| rpl_semi_sync_master_timeout       | 10000 |
| rpl_semi_sync_master_trace_level   | 32    |
| rpl_semi_sync_master_wait_no_slave | ON    |
+------------------------------------+-------+
4 rows in set (0.01 sec)
```


### 从节点安装semisync_slave

- 1、在从服务器的mysql服务器上执行如下命令

``` shell
mysql> install plugin rpl_semi_sync_slave soname 'semisync_slave.so';  安装模块
mysql> set global rpl_semi_sync_slave_enabled = 1;   启用半同步复制从节点
mysql> stop slave;
mysql> start slave;
mysql> show variables like '%semi%'; 查看设置是否成功

MariaDB [(none)]> show variables like '%sem%';
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| rpl_semi_sync_slave_enabled     | ON    |
| rpl_semi_sync_slave_trace_level | 32    |
+---------------------------------+-------+
2 rows in set (0.00 sec)
```

- 2、重启IO_THREAD

重启进程使其模块生效：`mysql> STOP SLAVE IO_THREAD; START SLAVE IO_THREAD;`


### 关闭从节点IO线程验证

关闭从节点`MYSQL> STOP SLAVE IO_THREAD;`

关闭从节点后，主节点修改数据，验证从节点超时，需要等待超时时间，过了之后才会加入数据，而且从节点未同步。
``` sql
MariaDB [imkindu]> insert into user values (6,'jianjian');
Query OK, 1 row affected (10.01 sec)    # 耗时10s，超时时长

MariaDB [imkindu]> select * from user;
+------+----------+
| id   | name     |
+------+----------+
|    1 | adu      |
|    2 | jianjian |
|    3 | dongdong |
|    4 | dongge   |
|    5 | aoteman  |
|    6 | jianjian |
+------+----------+
6 rows in set (0.00 sec)
```
开启从节点`MYSQL> START SLAVE IO_THREAD;`

开启从节点半同步复制，主节点添加数据，从节点立马同步。

``` sql
MariaDB [imkindu]> insert into user values (7,'haojian');
Query OK, 1 row affected (0.01 sec)
```

### 修改配置文件永久生效


在Master和Slave的my.cnf中编辑：

``` text  
# On Master  
[mysqld]  
rpl_semi_sync_master_enabled=1  
rpl_semi_sync_master_timeout=1000   #此单位是毫秒  
# On Slave  
[mysqld]  
rpl_semi_sync_slave_enabled=1  
```