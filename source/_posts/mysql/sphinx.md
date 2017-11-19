---
title : sphinx
date : 2016-11-09 20:30:00
author : imkindu
categories : 
- mysql
- database
tags : 
- sphinx
---

　　Sphinx是一个基于SQL的全文检索引擎，可以结合MySQL,PostgreSQL做全文搜索，它可以提供比数据库本身更专业的搜索功能，使得应用程序更容易实现专业化的全文检索。Sphinx特别为一些脚本语言设计搜索API接口，如PHP,Python,Perl,Ruby等，同时为MySQL也设计了一个存储引擎插件。

<!--more-->

## 全文索引两过程


**索引建立（Indexing）**

　　索引建立：将现实世界中所有的结构化和非结构化数据提取信息，创建索引的过程。

**搜索索引（Search）**

　　搜索索引：得到用户的查询请求，搜索创建的索引，然后返回结果的过程。



三个问题：
1、索引里面究竟存些什么？（Index）
2、如何创建索引？（Indexing）

    一些需要创建索引的文档（Documents）
    将原文档传给分词组件（Tokenizer）
    将得到的词元（Token）传给语言处理组件（Linguistic Processor）
    将得到的词（Team）传给索引组件（Indexer）


    文档频率：Document Frequency
        表示总共有多少文件包含此词（Term）

    词频率：Frequency
        此文件包含了几个此词（Term）

3、如何对索引进行搜索？（Search）

    1、用户输入查询语句
    2、对查询语句进行词法分析、语法分析、语言处理
    3、搜索索引，得到符号语法树的文档
    4、根据得到的文档和查询语句的相关性，对结果进行排序



## 安装sphinx


　　官网下载sphinx安装包  http://sphinxsearch.com/

Sphinx在mysql上的应用有两种方式：

　　1.采用API调用，如使用PHP、java等的API函数或方法查询。优点是可不必对mysql重新编译，服务端进程“低耦合”，且程序可灵活、方便的调用；缺点是如已有搜索程序的条件下，需修改部分程序。推荐程序员使用。

　　2.使用插件方式（sphinxSE）把sphinx编译成一个mysql插件并使用特定的sql语句进行检索。其特点是，在sql端方便组合，且能直接返回数据给客户端。不必二次查询,在程序上仅需要修改对应的sql，但这对使用框架开发的程序很不方便，比如使用了ORM。另外还需要对mysql进行重新编译，且需要mysql-5.1以上版本支持插件存储。


这里的安装主要介绍的是第一种通过api调用的方式。Sphinx的安装如下：

``` shell
#下载最新稳定版
wget http://www.sphinxsearch.com/downloads/sphinx-0.9.9.tar.gz
tar xzvf sphinx-0.9.9.tar.gz
cd sphinx-0.9.9
./configure --prefix=/usr/local/sphinx/   --with-mysql  --enable-id64
make
make install
注意：采用这种方式安装不支持中文分词。
```

bin 
    indexer
    search
    searchd  服务

etc

    配置文件

var
    形成的索引表


## 配置

    主数据源
        source main {
            type			= mysql
            sql_host		= localhost
            sql_user		= test
            sql_pass		=
            sql_db			= test
            sql_port		= 3306	# optional, default is 3306
            sql_query		= \         # 获取数据的sql语句
                SELECT id, group_id, UNIX_TIMESTAMP(date_added) AS date_added, title, content FROM documents
            sql_query_info  = select * from post where id=$id
        }

    增量数据源
        source delta:main{

        }

    主数据索引
        index main{

        }

    增量数据索引
        index delta:main{

        }

    分布式索引
        index dist1{

        }

    索引器
        indexer{

        }

    服务进程
        searchd {

        }


## 创建索引

    indexer
        -c          指定配置文件
        -all        对所有索引重新编制索引



## php-sphinx扩展

　　编译安装php sphinx扩展包

### libsphinxclient 

　　php模块需要

``` shell
# 在sphinx安装包中，包含的有libsphinxclient
cd sphinx-2.2.3/api/libsphinxclient  
./configure --prefix=/usr/local/sphinx  
make &&  make install  
```

### PHP Sphinx 扩展模块

``` shell
#如果没有扩展安装包，先下载  
wget http://pecl.php.net/get/sphinx-1.3.2.tgz  
tar zxvf sphinx-1.3.2.tgz  
cd sphinx-1.3.2  
phpize  
./configure --with-php-config=/usr/local/php/bin/php-config --with-sphinx=/usr/local/sphinx  
make  
make install  
```

### 修改php.ini

``` shell
[sphinx]  
extension=sphinx.so 
```

### sphinxapi.php

　　php-sphinx还提供了一种API方式的查询方法，在sphinx安装包文件中/api目录下，有一个`sphinxapi.php`文件，在使用php代码查询时，只需要引入这个文件即可。

## Sphinx中文分词

　　中文的全文检索和英文等latin系列不一样，后者是根据空格等特殊字符来断词，而中文是根据语义来分词。中文分词主要有2个插件

- 1.**Coreseek**

　　Coreseek是现在用的最多的sphinx中文全文检索，它提供了为Sphinx设计的中文分词包LibMMSeg ，是基于sphinx的基础上开发的。

- 2.**sfc(Sphinx-for-chinese)**

　　sfc（sphinx-for-chinese）是由网友happy兄提供的另外一个中文分词插件。其中文词典采用的是xdict。



### coreseek


　　Coreseek发布了3.2.14版本和4.1版本，其中的3.2.14版本是2010年发布的，它是基于Sphinx0.9.9搜索引擎的。而4.1版本是2011年发布的，它是基于Sphinx2.0.2的。Sphinx从0.9.9到2.0.2还是有改变了很多的，有很多功能，比如sql_attr_string等是在0.9.9上面不能使用的。所以在安装之前请判断清楚你需要安装的是哪个版本，在google问题的时候也要弄清楚这个问题的问题和答案是针对哪个版本的。我个人强烈建议使用4.1版本。


oreseek里有2个文件夹 一个是mmseg中文分词包 还有一个是csft(其实就是sphinx)包 都要安装


### 安装mmseg中文分词


``` shell
./configure --prefix=/usr/local/mmseg
```

编译时可能会报错config.status: error: cannot find input file: src/Makefile.in

通过automake来解决
首先检查是否安装了libtool如果没有 
yum -y install libtool
automake
如果automake报错 原因可能是下列

``` shell
Libtool library used but `LIBTOOL' is undefined
The usual way to define `LIBTOOL' is to add `AC_PROG_LIBTOOL'
to `configure.ac' and run `aclocal' and `autoconf' again.
If `AC_PROG_LIBTOOL' is in `configure.ac', make sure
its definition is in aclocal's search path.
```

如果以上步骤都没成功，那么试下以下办法（把下面的命令都执行一遍，就好了）

``` shell
aclocal
libtoolize --force
automake --add-missing
autoconf
autoheader
make clean
```

### 编译安装csft-4.1

``` shell

./buildconf.sh 

./configure 
--prefix=/usr/local/coreseek --with-mysql-includes=/usr/includes/mysql --with-mysql-libs=/usr/lib64/mysql/ 
--with-mmseg=/usr/local/mmseg --with-mmseg-includes=/usr/local/mmseg/include/mmseg/ --with-mmseg-libs=/usr/local/mmseg/lib/
```

yum安装的mysql的include和libs文件夹一般是安装在/usr/include/mysql和/usr/lib64/mysql下面

所以这里的--with-mysql可以使用--with-mysql-includes和--with-mysql-libs来进行替换。

在buildconf.sh 时报错，以下网站有提示

http://blog.csdn.net/jcjc918/article/details/39032689



## 实时索引


http://www.wuzexin.cn/post-58.html