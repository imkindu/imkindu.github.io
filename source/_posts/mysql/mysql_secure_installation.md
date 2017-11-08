---
title: mysql_secure_installation
date: 2017-09-22 09:00:00
author: imkindu
categories: mysql
tags:
- mysql安全
- mysql初始化
---

　　mysql安装完成后，系统自带了几个用户和匿名用户可以登录，但是为了安全考虑，我们需要执行一遍`mysql_secure_installation`安全向导初始化一下我们的数据库。

<!--more-->

```
MariaDB [mysql]> select User,Host,Password from user;
+------+-----------+----------+
| User | Host      | Password |
+------+-----------+----------+
| root | localhost |          |
| root | centos69  |          |
| root | 127.0.0.1 |          |
| root | ::1       |          |
|      | localhost |          |
|      | centos69  |          |
+------+-----------+----------+
6 rows in set (0.00 sec)
```

　　刚安装完mysql，我们直接执行mysql命令就可以登录，默认使用的是root@localhost 空密码登录的，安全的第一件事就是给root这个超级霸主设置密码，使用mysql_secure_installation可以初始化让我们设置密码，还能限制root从远程登录。



### 文件位置

　　初始化脚本mysql_secure_installation放在哪里呢？默认在`安装目录下的/bin`目录下。

```
> /usr/local/mysql/bin/mysql_secure_installation
```

### 初始化

　　再来看一下这个脚本帮我们做了什么事，我们执行一遍


```
[ root@centos69 bin ]# ./mysql_secure_installation 
./mysql_secure_installation: line 393: find_mysql_client: command not found
NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!
In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.
####################################################################
Enter current password for root (enter for none): 				# 询问当前root密码，默认为空
OK, successfully used password, moving on...
Setting the root password ensures that nobody can log into the MariaDB
root user without the proper authorisation.
####################################################################
Set root password? [Y/n]  Y							# 设置root密码
New password: 
Re-enter new password: 
Password updated successfully!
Reloading privilege tables..
 ... Success!
By default, a MariaDB installation has an anonymous user, allowing anyone
to log into MariaDB without having to have a user account created for
them.  This is intended only for testing, and to make the installation
go a bit smoother.  You should remove them before moving into a
production environment.
####################################################################
Remove anonymous users? [Y/n] y 						# 移除匿名账号
 ... Success!
Normally, root should only be allowed to connect from 'localhost'.  This
ensures that someone cannot guess at the root password from the network.
####################################################################
Disallow root login remotely? [Y/n] n 						# 禁止root远程登录
 ... skipping.
By default, MariaDB comes with a database named 'test' that anyone can
access.  This is also intended only for testing, and should be removed
before moving into a production environment.
####################################################################
Remove test database and access to it? [Y/n] n 					# 删除test数据库
 ... skipping.
Reloading the privilege tables will ensure that all changes made so far
will take effect immediately.
####################################################################
Reload privilege tables now? [Y/n] y 						# 刷新授权表
 ... Success!
Cleaning up...
All done!  If you've completed all of the above steps, your MariaDB
installation should now be secure.
Thanks for using MariaDB!
```

---

### 效果

<div align="center">
![](/images/mysql/001.gif)
</div>