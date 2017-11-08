---
title: 编译安装mariadb-5.5.57
date: 2017-09-21 20:00:00
author: imkindu
categories: mysql
tags:
- 编译安装mariadb
---


　　编译安装mariadb-5.5.57，mysql5.5之后mysql编译基于cmake，所以需要填编译安装cmake.

<!--more-->



　　cmake的重要特性之一是其独立于源码(out-of-source)的编译功能，即编译工作可以在另一个指定的目录中而非源码目录中进行，这可以保证源码目录不受任何一次编译的影响，因此在同一个源码树上可以进行多次不同的编译，如针对于不同平台编译。





## 安装cmake

　　跨平台编译器

```
# tar xf cmake-2.8.8.tar.gz
# cd cmake-2.8.8
# ./bootstrap
# gmake 
# gmake install
```


### cmake查看编译选项

```
cmake . -LH 			# . 表示当前目录  -LH 编译完后打印出选项
```

cmake指定编译选项的方式不同于make，其实现方式对比如下：

```
./configure           	cmake .
./configure --help    	cmake . -LH  or  ccmake .
```

## 编译


### 查看编译选项

```
[ root@centos50 mariadb-5.5.57 ]# cmake . -LH
-- Running cmake version 2.8.8
-- MariaDB 5.5.57
-- Packaging as: mariadb-5.5.57-Linux-x86_64
-- Could NOT find Curses (missing:  CURSES_LIBRARY CURSES_INCLUDE_PATH) 
CMake Error at cmake/readline.cmake:85 (MESSAGE):
  Curses library not found.  Please install appropriate package,

      remove CMakeCache.txt and rerun cmake.On Debian/Ubuntu, package name is libncurses5-dev, on Redhat and derivates it is ncurses-devel.
Call Stack (most recent call first):
  cmake/readline.cmake:196 (FIND_CURSES)
  CMakeLists.txt:351 (MYSQL_CHECK_READLINE)


-- Configuring incomplete, errors occurred!
-- Cache values
// Choose the type of build, options are: None(CMAKE_CXX_FLAGS or CMAKE_C_FLAGS used) Debug Release RelWithDebInfo MinSizeRel
CMAKE_BUILD_TYPE:STRING=RelWithDebInfo
// install prefix
CMAKE_INSTALL_PREFIX:PATH=/usr/local/mysql 								# 编译安装位置
// Set to true if this is a community build
COMMUNITY_BUILD:BOOL=ON 												# 社区
// Enable profiling
ENABLED_PROFILING:BOOL=ON 												# Query 诊断分析工具
// Enable gcov (debug, Linux builds only)
ENABLE_GCOV:BOOL=OFF 													# 是否包含 Gcov 支持
// Installation directory layout. Options are: STANDALONE (as in zip or tar.gz installer) RPM DEB SVR4
INSTALL_LAYOUT:STRING=STANDALONE 										# 选择预定义的安装
// default MySQL data directory
MYSQL_DATADIR:PATH=/usr/local/mysql/data 									# 数据存放位置
// Allow linking with GPLv2-incompatible system libraries. Only set it you never plan to distribute the resulting binaries
NOT_FOR_DISTRIBUTION:BOOL=OFF
// PATH to MySQL TMP dir. Defaults to the P_tmpdir macro in <stdio.h>
TMPDIR:PATH= 															# 临时目录
// Enable address sanitizer
WITH_ASAN:BOOL=OFF
// Options are: none complex all
WITH_EXTRA_CHARSETS:STRING=all 											# 额外的字符集，包括  all
// Build with jemalloc (possible values are 'yes', 'no', 'auto')
WITH_JEMALLOC:STRING=auto
// Compile with tcp wrappers support
WITH_LIBWRAP:BOOL=OFF
// Use bundled readline
WITH_READLINE:BOOL=OFF 													# 使用捆绑的readline
// Use safemalloc memory debugger. Will result in slower execution. Options are: ON OFF AUTO.
WITH_SAFEMALLOC:STRING=AUTO
// Options are: no bundled yes(prefer os library if present otherwise use bundled) system(use os library)
WITH_SSL:STRING=no 														# 是否支持SSL
// Compile MySQL with unit tests
WITH_UNIT_TESTS:BOOL=ON
// Valgrind instrumentation
WITH_VALGRIND:BOOL=OFF
// Which zlib to use (possible values are 'bundled' or 'system')
WITH_ZLIB:STRING=system 													# 是否支持Zlib
```



## 编译安装mariadb


### 安装依赖包

```shell
yum install ncurses-devel
```

### 添加用户

```shell
# groupadd -r mysql
# useradd -g mysql -r -d /mydata/data mysql
```

### 编译

```shell
# tar xf mysql-5.5.33.tar.gz 
# cd mysql-5.5.33
# cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mariadb-5.5/ \
-DMYSQL_DATADIR=/usr/local/mariadb-5.5/data \
-DSYSCONFDIR=/etc \
-DMYSQL_USER=mysql \
-DMYSQL_TCP_PORT=3306 \
-DWITH_INNOBASE_STORAGE_ENGINE=1 \
-DWITH_ARCHIVE_STORAGE_ENGINE=1 \
-DWITH_BLACKHOLE_STORAGE_ENGINE=1 \
-DWITH_READLINE=1 \
-DWITH_SSL=system \
-DWITH_SSL=bundled \
-DWITH_ZLIB=system \
-DWITH_LIBWRAP=0 \
-DMYSQL_UNIX_ADDR=/tmp/mysql.sock \
-DDEFAULT_CHARSET=utf8 \
-DDEFAULT_COLLATION=utf8_general_ci
-DWITH_DEBUG=0 \
-DEXTRA_CHARSETS=all \
# make 
# make install
```



如果想清理此前的编译所生成的文件，则需要使用如下命令：

```shell
make clean
rm CMakeCache.txt
```


## 编译选项解释


> 指定安装文件的安装路径时常用的选项：

```shell
-DCMAKE_INSTALL_PREFIX=/usr/local/mysql
-DMYSQL_DATADIR=/data/mysql
-DSYSCONFDIR=/etc
```


> 默认编译的存储引擎包括：csv、myisam、myisammrg和heap。若要安装其它存储引擎，可以使用类似如下编译选项：

```shell
-DWITH_INNOBASE_STORAGE_ENGINE=1
-DWITH_ARCHIVE_STORAGE_ENGINE=1
-DWITH_BLACKHOLE_STORAGE_ENGINE=1
-DWITH_FEDERATED_STORAGE_ENGINE=1
```


> 若要明确指定不编译某存储引擎，可以使用类似如下的选项：

```shell
-DWITHOUT_<ENGINE>_STORAGE_ENGINE=1
比如：
-DWITHOUT_EXAMPLE_STORAGE_ENGINE=1
-DWITHOUT_FEDERATED_STORAGE_ENGINE=1
-DWITHOUT_PARTITION_STORAGE_ENGINE=1
```

>如若要编译进其它功能，如SSL等，则可使用类似如下选项来实现编译时使用某库或不使用某库：

```shell
-DWITH_READLINE=1
-DWITH_SSL=system
-DWITH_ZLIB=system
-DWITH_LIBWRAP=0
```

> 其它常用的选项：

```shell
-DMYSQL_TCP_PORT=3306
-DMYSQL_UNIX_ADDR=/tmp/mysql.sock
-DENABLED_LOCAL_INFILE=1
-DEXTRA_CHARSETS=all
-DDEFAULT_CHARSET=utf8
-DDEFAULT_COLLATION=utf8_general_ci
-DWITH_DEBUG=0
-DENABLE_PROFILING=1
```



---

## 编译安装后续步骤同二进制安装



- 初始化数据库
- 配置文件my.cnf
- mysql环境变量
- service服务脚本


