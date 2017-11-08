---
title: 二进制安装mariadb-5.5.57
date: 2017-09-21 20:00:00
author: imkindu
categories: mysql
tags:
- mysql二进制安装
---


　　mariadb官方网站上提供了三种不同形式的程序包：源码包版、程序包管理器版、和二进制版，如下图所示。二进制版是由官方编译好的绿色版，相比源码包版安装更简单，比起程序包管理器版又多一点自由度，算是二者的折中方案。另外要注意它依赖于glibc，需要注意glibc的版本。

<!--more-->

---

## 准备

- 1、确认glibc

```
[ root@centos69 etc ]# rpm -qa glibc
glibc-2.12-1.209.el6.x86_64
```

- 2、关闭iptables和selinux

```
[ root@centos69 etc ]# getenforce
Permissive
[ root@centos69 etc ]# service iptables status
iptables: Firewall is not running.
```


- 3、创建mysql系统用户

```
useradd -r mysql
```

- 4、下载二进制mariadb包

[点击进入mariadb官网](https://downloads.mariadb.org/mariadb/5.5.57/#)

<div align="center">
![](/images/mysql001.png)
</div>

- 5、解压至目录/usr/local/

/usr/local/mysql是默认指定安装目录

```
tar -xvf mariadb-5.5.57-linux-x86_64.tar.gz -C /usr/local/
```

- 6、创建软链接mysql

```
ln -s mariadb-5.5.57/ mysql
drwxr-xr-x. 12 root root 4096 Sep 11 12:58 mariadb-5.5.57
lrwxrwxrwx.  1 root root   15 Sep 11 12:58 mysql -> mariadb-5.5.57/
```


---

## 安装

　　mysql的进程默认是使用mysql用户，所以mysql目录以及数据库目录data都应该给与mysql用户对应权限


- 1、修改目录属主属组

```
[ root@centos69 mysql ]# chown -R mysql:mysql /usr/local/mysql/
```

- 2、创建数据库目录，如果不单独指定则默认使用mysql下面的data目录

```
[ root@centos69 mysql ]# mkdir -p /var/mysql/data
```

- 3、更改数据库目录的属主属组

```
[ root@centos69 mysql ]# chown -R mysql:mysql /var/mysql/data
```

- 4、安装数据库

　　初始化数据库，脚本在scripts目录下的mysql_install_db

```
[ root@centos69 data ] scripts/mysql_install_db --user=mysql --datadir=/var/mysql/data/
```

　　查看数据库存放目录，一个数据库就是一个目录

```
[ root@centos69 mysql ]# ll /var/mysql/data/
total 28744
drwx------. 2 mysql mysql     4096 Sep 11 13:13 mysql
drwx------. 2 mysql mysql     4096 Sep 11 13:13 performance_schema
drwx------. 2 mysql mysql     4096 Sep 11 13:13 test
```

---

## 配置


　　二进制安装后，需要自己添加service脚本、chkconfig开机启动、man帮助手册等

- 1、将bin目录路径导入PATH环境变量

```
vim /etc/profile.d/mysql.sh
export PATH=/usr/local/mysql/bin:$PATH
# 启动生效
exec bash
```
-  2、创建头文件符号链接

```
cd /usr/local/include/
ln -s ../mysql/include/mysql/ mysql
```

- 3、将man路径导入系统man手册

```
vim /etc/man.config
MANPATH /usr/local/mysql/man
```

- 4、服务脚本拷贝至/etc/rc.d/init.d

```
cd /usr/local/mysql
cp support-files/mysql.server /etc/rc.d/init.d/mysqld
chkconfig --add mysqld
```

- 5、复制模板配置文件至/etc/目录


　　通用二进制格式安装的服务程序其配置文件查找次序：

> `/etc/my.cnf  --> /etc/mysql/my.cnf  --> --default-extra-file=/PATH/TO/CONF_FILE  --> ~/.my.cnf`

```
cp support-files/my-large.cnf /etc/my.cnf
```


- 6、更改配置文件，关闭域名反解

```
vim /etc/my.cnf
[mysqld]后面加一句
skip-name-resolve=TRUE
```

- 7、更改配置文件，指定data目录

```
[mysqld]
datadir=/var/mysql/data
```


---

## 启动

> 启动mysql查看进程

```
[ root@centos69 ~ ]# service mysqld restart
Shutting down MySQL... SUCCESS! 
Starting MySQL.170911 14:29:57 mysqld_safe Logging to '/var/mysql/data/centos69.err'.
170911 14:29:58 mysqld_safe Starting mysqld daemon with databases from /var/mysql/data
. SUCCESS! 
```
```
[ root@centos69 data ]# service mysqld status
 SUCCESS! MySQL running (11118)
[ root@centos69 data ]# netstat -antp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address               Foreign Address             State       PID/Program name   
tcp        0      0 0.0.0.0:3306                0.0.0.0:*                   LISTEN      11118/mysqld        
tcp        0      0 0.0.0.0:111                 0.0.0.0:*                   LISTEN      1466/rpcbind        
tcp        0      0 0.0.0.0:37941               0.0.0.0:*                   LISTEN      1523/rpc.statd      
tcp        0      0 0.0.0.0:22                  0.0.0.0:*                   LISTEN      1754/sshd           
tcp        0      0 127.0.0.1:25                0.0.0.0:*                   LISTEN      1844/master         
tcp        0      0 172.18.56.69:22             172.18.252.123:53848        ESTABLISHED 10704/sshd          
tcp        0      0 172.18.56.69:22             172.18.252.123:51284        ESTABLISHED 6298/sshd           
tcp        0      0 :::111                      :::*                        LISTEN      1466/rpcbind        
tcp        0      0 :::56402                    :::*                        LISTEN      1523/rpc.statd      
tcp        0      0 :::22                       :::*                        LISTEN      1754/sshd           
tcp        0      0 :::23                       :::*                        LISTEN      1765/xinetd         
tcp        0      0 ::1:25                      :::*                        LISTEN      1844/master 
```

> 匿名登录mysql

```
[ root@centos69 ~ ]# mysql
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 2
Server version: 5.5.57-MariaDB MariaDB Server
Copyright (c) 2000, 2017, Oracle, MariaDB Corporation Ab and others.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
MariaDB [(none)]> 
```