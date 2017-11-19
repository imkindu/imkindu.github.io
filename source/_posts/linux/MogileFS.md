---
title : MongileFS分布式文件存储系统
date : 2017-11-11 09:00:00
author : imkindu
categories :
- linux
tags :
- MongileFS
---


　　MogileFS是一个开源的分布式文件存储系统，由LiveJournal旗下的Danga Interactive公司开发。Danga团队开发了包括 Memcached、MogileFS、Perlbal 等多个知名的开源项目。目前使用MogileFS 的公司非常多，如日本排名先前的几个互联公司及国内的yupoo(又拍)、digg、豆瓣、1号店、大众点评、搜狗和安居客等，分别为所在的组织或公司管理着海量的图片。

<!--more-->



MogileFS由3个部分组成：
(1)server：主要包括mogilefsd和mogstored两个应用程序。mogilefsd实现的是tracker，它通过数据库来保存元数据信息，包括站点domain、class、host等；mogstored是存储节点(store node)，它其实是个WebDAV服务，默认监听在7500端口，接受客户端的文件存储请求。在MogileFS安装完后，要运行mogadm工具将所有的store node注册到mogilefsd的数据库里，mogilefsd会对这些节点进行管理和监控。
(2) utils（工具集）：主要是MogileFS的一些管理工具，例如mogadm等。
(3)客户端API：MogileFS的客户端API很多，例如Perl、PHP、Java、Python等，用这个模块可以编写客户端程序，实现文件的备份管理功能等。

初始化数据库
    mogdbsetup --help


tracker节点
    修改配置文件
        vim /etc/mogilefs/mogilefsd.conf

