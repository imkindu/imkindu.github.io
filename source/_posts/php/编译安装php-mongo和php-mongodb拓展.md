---
title: php-mongo和php-mongodb扩展
date: 2017-10-11 20:00:00
author: imkindu
categories: php
tags:
- php-mongo
- php-mongodb
---

　　php要使用mongodb需要安装php对应的mongo扩展包，这一篇将介绍php-mongo和php-mongodb的相关简介和差别。并编译安装两个拓展。

<!--more-->

---

## php-mongo和php-mongodb差别

　　php-mongo和php-mongodb都是php对应mongo的扩展接口，在原本的php5.6等版本上，我们都是习惯用的php-mongo这个扩展，使用语法简单，但是在最新的PHP7版本上，现在都是使用的php-mongodb这个扩展，同时还加入了mongodb新版的特性。

　　php-mongo这个扩展现在已经废弃了，不过bug和security 方面的问题还会继续修复，这个可以查看pecl的官网信息。

　　[`http://pecl.php.net/package/mongo`](http://pecl.php.net/package/mongo)  			[`http://pecl.php.net/package/mongodb`](http://pecl.php.net/package/mongodb)

<div align="center">
![](/images/php/01.png)
</div>

<div align="center">
![](/images/php/02.png)
</div>

<div align="center">
![](/images/php/03.png)
</div>

---

## 安装PHP-mongo扩展


> 1.首先上http://pecl.php.net上面搜索mongo,得到下载地址

```shell
wget http://pecl.php.net/get/mongo-1.6.11.tgz
tar zxvf ./mongo-1.6.11.tgz
```

> 2.解压进入,phpize后进行编译

```shell
cd ./mongo-1.6.11
phpize #有可能要写全phpize的地址
./configure --with-php-config=/usr/local/php/bin/php-config
make && make install
```

> 3.编译成功后出现:

```shell
[ root@centos50 mongo-1.6.16 ]# make install
Installing shared extensions:     /usr/local/php5.6/lib/php/extensions/no-debug-zts-20131226/
```

> 4.得其地址写入php.ini

```shell
extension = mongo.so #有可能要写全mongo.so的路径,也就是上面的提示
```
> 5.安装完以后,看phpinfo()中有没有这个扩展,有就表示安装成功;


---

## 编译安装PHP-mongodb扩展


> 1.首先上http://pecl.php.net上面搜索mongo,得到下载地址

```shell
wget http://pecl.php.net/get/mongodb-1.3.11.tgz
tar zxvf ./mongodb-1.3.11.tgz
```

> 2.解压进入,phpize后进行编译

```shell
cd ./mongodb-1.3.11
phpize #有可能要写全phpize的地址
./configure --with-php-config=/usr/local/php/bin/php-config
make && make install
```

> 3.编译成功后出现:

```shell
[ root@centos50 mongo-1.3.16 ]# make install
Installing shared extensions:     /usr/local/php5.6/lib/php/extensions/no-debug-zts-20131226/
```

> 4.得其地址写入php.ini

```shell
extension = mongodb.so #有可能要写全mongodb.so的路径,也就是上面的提示
```
> 5.安装完以后,看phpinfo()中有没有这个扩展,有就表示安装成功;


