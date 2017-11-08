---
title: ldirectord高可用负载
date: 2017-10-21 19:00:00
author: imkindu
categories: linux
tags:
- lvs
- ipvsadm
- ldirectord
---


　　为了从主Director将LVS负载均衡资源故障转移到备用Director，并从集群中自动移除节点，我们需要使用ldirectord程序，这个程序在启动时自动建立IPVS表，然后监视集群节点的健康情况，在发现失效节点时将其自动从IPVS表中移除。

<!--more-->

## 工作原理

　　ldirectord守护进程通过向每台真实服务器真实IP（RIP）上的集群资源发送访问请求来实现对真实服务器的监控，这对所有类型的LVS集群都是成立的：LVS-DR，LVS-NAT和LVS-TUN。正常情况下，为每个Director上的VIP地址运行一个ldirectord守护进程，当真实服务器不响应运行在Director上的ldirectord守护进程时，ldirectord守护进程运行适当的ipvsadm命令将VIP地址从IPVS表中移除。（以后，当真实服务器回到在线状态时，ldirectord使用适当的ipvsadm命令将真实服务器重新添加到IPVS表中）

　　为了监视web集群内的真实服务器，ldirectord守护进程使用HTTP协议向每个真实服务器请求一个专用的web页面，如果真实服务器是健康的，Director知道将从真实服务器接收到什么内容，如果从真实服务器返回应答字串或web页面的时间太长，或根本没有返回任何内容，或返回的内容不是预期的，Director就知道该真实服务器出错了，并从IPVS表中将这个真实服务器移除。



## 准备

### 下载ldirectord

[ldirectord-centos 6版本下载地址](http://download.opensuse.org/repositories/network:/ha-clustering:/Stable/CentOS_CentOS-6/x86_64/) http://download.opensuse.org/repositories/network:/ha-clustering:/Stable/CentOS_CentOS-6/x86_64/

### 安装

因为ldirectord依赖其他安装包，最好使用yum安装解决依赖性

```shell
yum localinstall ldirectord-centos6-3.9.6-0rc1.1.1.x86_64.rpm 
```

### 查看包列表文件

```
[ root@centos10x ~ ]# rpm -ql ldirectord 
/etc/ha.d
/etc/had.d/ldirectord.cf 			# 主配置文件，需自己添加，可以拷贝配置模板文件
/etc/ha.d/resource.d
/etc/ha.d/resource.d/ldirectord
/etc/init.d/ldirectord 			# 服务脚本
/etc/logrotate.d/ldirectord
/usr/lib/ocf/resource.d/heartbeat/ldirectord
/usr/sbin/ldirectord 				# 命令
/usr/share/doc/ldirectord-3.9.6
/usr/share/doc/ldirectord-3.9.6/COPYING
/usr/share/doc/ldirectord-3.9.6/ldirectord.cf 	   # 模板配置文件 
/usr/share/man/man8/ldirectord.8.gz
```


## 配置

　　ldirectord的主要配置文件在`ldirectord.cf`文件

### 全局配置

```shell
[ root@centos10x ha.d ]# cat ldirectord.cf 
# Global Directives
checktimeout=3　							   # 超时时间
checkinterval=1 							   # 检测间隔时间 
#fallback=127.0.0.1:80 						# 当RS全部故障，由此生效
#fallback6=[::1]:80
autoreload=yes 								# 修改配置文件自动生效
#logfile="/var/log/ldirectord.log" 			# 日志文件
#logfile="local0"
#emailalert="admin@x.y.z"
#emailalertfreq=3600
#emailalertstatus=all
quiescent=no
```

### 简单http虚拟服务配置示例 

```shell
# Sample for an http virtual service 		# 简单http虚拟服务器示例
virtual=172.18.56.100:80
	real=172.18.56.50:80 gate 1 			 # real server : gate表示dr模型 后面接数字表示 权重
	real=172.18.56.51:80 gate 1
	fallback=127.0.0.1:80 gate
	service=http
	scheduler=rr 							# 调度算法
	#persistent=600
	#netmask=255.255.255.255
	protocol=tcp  						   # 协议
	checktype=negotiate
	checkport=80 							# 检测端口
	request="index.html" 					# 检测文件
	receive="Test Page" 					 # 检测回应内容
	virtualhost=www.x.y.z 				   # 虚拟网址
```
**重启服务**

```shell
[ root@centos10x ha.d ]# service ldirectord restart
Restarting ldirectord... success
[ root@centos10x ha.d ]# service ldirectord status
ldirectord for /etc/ha.d/ldirectord.cf is running with pid: 3941
```

**检查**

```shell
[ root@centos10x ha.d ]# ipvsadm -L -n
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  172.18.56.100:80 wrr
  -> 172.18.56.50:80              Route   1      0          2         
  -> 172.18.56.51:80              Route   1      0          3   
```

```shell
imkindu@ubuntu:~$ curl 172.18.56.100 
192.168.56.50
imkindu@ubuntu:~$ curl 172.18.56.100 
192.168.56.51
```

---

## fallback自动断开恢复

　　ldirectord当某台real server断开，或者全部断开后，会自动从ipvsadm列表中剔除，当恢复时，自动加入。全部断开时，会返回配置的fallback页面。 


### 断开Real server1

```shell
[ root@centos50 www ]# service httpd stop
Stopping httpd:                                            [  OK  ]
```

### 查看directory状态 

```shell
[ root@centos10x ha.d ]# ipvsadm -L -n
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  172.18.56.100:80 wrr
  -> 172.18.56.51:80              Route   1      0          0  
```

　　real server1会自动从ipvsadm中去除

### 去除所有的real server,查看directory情况 

```shell
[ root@centos10x ha.d ]# ipvsadm -L -n
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  172.18.56.100:80 wrr
  -> 127.0.0.1:80                 Local   1      0          0 
```

### 此时客户端请求，返回的就是fallback

```shell
192.168.56.51
imkindu@ubuntu:~$ curl 172.18.56.100 
192.168.56.10
```

## mysql集群环境配置

```shell
#Sample configuration for a MySQL virtual service.
#virtual = 192.168.10.74:3306
#	real=sql01->sql03:3306 gate 10
#	fallback=127.0.0.1:3306
#	service=mysql
#	scheduler=wrr
#	#persistent=600
#	#netmask=255.255.255.255
#	protocol=tcp
#	checktype=negotiate
#	login="readuser"
#	passwd="genericpassword"
#	database="portal"
#	request="SELECT * FROM link"
```