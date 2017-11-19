---
title : proxysql
date : 2017-11-15 21:30:00
author : imkindu
categories :
- mysql
- database
tags : 
- proxysql
---



http://www.proxysql.com/

<!--more-->

[root@centos80 ~]# wget https://github.com/sysown/proxysql/releases/download/v1.4.3/proxysql-1.4.3-1-centos7.x86_64.rpm

[root@centos80 ~]# yum install ./proxysql-1.4.2-1-centos7.x86_64.rpm 
Loaded plugins: fastestmirror, langpacks
Examining ./proxysql-1.4.2-1-centos7.x86_64.rpm: proxysql-1.4.2-1.x86_64
Marking ./proxysql-1.4.2-1-centos7.x86_64.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package proxysql.x86_64 0:1.4.2-1 will be installed
--> Finished Dependency Resolution
base                                                                                                           | 3.6 kB  00:00:00     
epel                                                                                                           | 4.3 kB  00:00:00     

Dependencies Resolved

======================================================================================================================================
 Package                   Arch                    Version                    Repository                                         Size
======================================================================================================================================
Installing:
 proxysql                  x86_64                  1.4.2-1                    /proxysql-1.4.2-1-centos7.x86_64                   19 M

Transaction Summary
======================================================================================================================================
Install  1 Package

Total size: 19 M
Installed size: 19 M
Is this ok [y/d/N]: y
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : proxysql-1.4.2-1.x86_64                                                                                            1/1 
  Verifying  : proxysql-1.4.2-1.x86_64                                                                                            1/1 

Installed:
  proxysql.x86_64 0:1.4.2-1                                                                                                           

Complete!






[root@centos80 ~]# rpm -ql proxysql
/etc/init.d/proxysql
/etc/proxysql.cnf               # 配置文件
/usr/bin/proxysql
/usr/share/proxysql/tools/proxysql_galera_checker.sh
/usr/share/proxysql/tools/proxysql_galera_writer.pl



[root@centos80 etc]# mysql -uadmin -padmin -h127.0.0.1 -P 6032
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
Server version: 5.5.30 (ProxySQL Admin Module)

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]> show databases;
+-----+---------+-------------------------------+
| seq | name    | file                          |
+-----+---------+-------------------------------+
| 0   | main    |                               |
| 2   | disk    | /var/lib/proxysql/proxysql.db |
| 3   | stats   |                               |
| 4   | monitor |                               |
+-----+---------+-------------------------------+
4 rows in set (0.00 sec)

MySQL [(none)]> 
