---
title : mysql备份还原
date : 2017-11-14 09:00:00
author : imkindu
categories :
- database
tags :
- mysqldump
- mysql备份恢复
---


mysql备份还原

<!--more-->

## 备份考量点

- 能容忍最多丢失多少数据
- 恢复数据需要多长时间完成
- 需要恢复哪些数据

## 备份类型

- 完全备份：整个数据集
- 部分备份：只备份数据子集

根据时间轴也可分为

- 完全备份
- 增量备份：仅备份最近一次完全备份或增量备份之后修改的数据
- 差异备份：仅备份最近一次完全备份以来变化的数据（最近一次完全备份）

<div align="center">
![](/images/mysql/bf.png)
</div>

根据备份时服务是否在线分为

- 热备：备份时，线下服务读写操作均可执行
- 温备：读操作可执行，但写操作不行
- 冷备：读写操作均不能执行

## 备份什么

- 数据
- 二进制日志、InnoDB事务日志
- 代码（存储过程、存储函数、触发器、调度器）
- 服务器的配置文件


## 备份工具

- mysqldump ：逻辑备份工具，使用所有存储引擎，对innodb支持热备
- cp/tar ：物理备份，冷备，使用所有存储引擎
- xtrabackup ：由percona提供的支持对Innodb做热备的工具，完全备份、增量备份



## mysqldump

将schema和数据存储在一起，形成一个巨大的sql语句的备份文件


对MyISAM 支持温备：需要锁定备份库 --lock-all-tables / --lock-tables


- --master-data 将binlog的位置和文件名追加到输出文件,方便以二进制日志恢复数据

导出的sql文件会指明备份到二进制日志哪个节点

``` sql
-- CHANGE MASTER TO MASTER_LOG_FILE='binlog.000001', MASTER_LOG_POS=245;
```

``` shell
[root@centos80 ~]# mysqldump -uroot -p --databases imkindu --master-data=2 > /app/mysql/backup/imkindu-17-11-14.sql 
Enter password: 
[root@centos80 ~]# 
[root@centos80 ~]# 
[root@centos80 ~]# 
[root@centos80 ~]# cat /app/mysql/backup/imkindu-17-11-14.sql 
-- MySQL dump 10.14  Distrib 5.5.52-MariaDB, for Linux (x86_64)
--
-- Host: localhost    Database: imkindu
-- ------------------------------------------------------
-- Server version	5.5.52-MariaDB

/*!40101 SET @OLD_CHARACTER_SET_CLIENT=@@CHARACTER_SET_CLIENT */;
/*!40101 SET @OLD_CHARACTER_SET_RESULTS=@@CHARACTER_SET_RESULTS */;
/*!40101 SET @OLD_COLLATION_CONNECTION=@@COLLATION_CONNECTION */;
/*!40101 SET NAMES utf8 */;
/*!40103 SET @OLD_TIME_ZONE=@@TIME_ZONE */;
/*!40103 SET TIME_ZONE='+00:00' */;
/*!40014 SET @OLD_UNIQUE_CHECKS=@@UNIQUE_CHECKS, UNIQUE_CHECKS=0 */;
/*!40014 SET @OLD_FOREIGN_KEY_CHECKS=@@FOREIGN_KEY_CHECKS, FOREIGN_KEY_CHECKS=0 */;
/*!40101 SET @OLD_SQL_MODE=@@SQL_MODE, SQL_MODE='NO_AUTO_VALUE_ON_ZERO' */;
/*!40111 SET @OLD_SQL_NOTES=@@SQL_NOTES, SQL_NOTES=0 */;

--
-- Position to start replication or point-in-time recovery from
--

-- CHANGE MASTER TO MASTER_LOG_FILE='binlog.000001', MASTER_LOG_POS=245;

--
-- Current Database: `imkindu`
--

CREATE DATABASE /*!32312 IF NOT EXISTS*/ `imkindu` /*!40100 DEFAULT CHARACTER SET latin1 */;

USE `imkindu`;

--
-- Table structure for table `user`
--

DROP TABLE IF EXISTS `user`;
/*!40101 SET @saved_cs_client     = @@character_set_client */;
/*!40101 SET character_set_client = utf8 */;
CREATE TABLE `user` (
  `id` int(11) DEFAULT NULL,
  `name` varchar(60) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
/*!40101 SET character_set_client = @saved_cs_client */;

--
-- Dumping data for table `user`
--

LOCK TABLES `user` WRITE;
/*!40000 ALTER TABLE `user` DISABLE KEYS */;
INSERT INTO `user` VALUES (1,'adu'),(2,'jianjian');
/*!40000 ALTER TABLE `user` ENABLE KEYS */;
UNLOCK TABLES;
/*!40103 SET TIME_ZONE=@OLD_TIME_ZONE */;

/*!40101 SET SQL_MODE=@OLD_SQL_MODE */;
/*!40014 SET FOREIGN_KEY_CHECKS=@OLD_FOREIGN_KEY_CHECKS */;
/*!40014 SET UNIQUE_CHECKS=@OLD_UNIQUE_CHECKS */;
/*!40101 SET CHARACTER_SET_CLIENT=@OLD_CHARACTER_SET_CLIENT */;
/*!40101 SET CHARACTER_SET_RESULTS=@OLD_CHARACTER_SET_RESULTS */;
/*!40101 SET COLLATION_CONNECTION=@OLD_COLLATION_CONNECTION */;
/*!40111 SET SQL_NOTES=@OLD_SQL_NOTES */;

-- Dump completed on 2017-11-14 10:37:54
```

### 二进制日志备份恢复

在增量备份恢复时，可以使用二进制日志从指定位置查看，并将后面的sql语句导出出来，进行备份恢复

从指定位置查看，并保持至文件，以便导入恢复
``` shell
[root@centos80 backup]# mysqlbinlog -j 245 binlog.000001 > /app/mysql/backup/binlog.000001-245.sql
```

在需要恢复的服务器上导入备份文件

``` shell
[root@centos81 ~]# mysql -uroot -p < binlog.000001-245.sql 
```

## xtrabackup

https://www.percona.com/downloads/XtraBackup/LATEST/

[root@centos80 ~]# yum install ./percona-xtrabackup-2.3.2-1.el7.x86_64.rpm 