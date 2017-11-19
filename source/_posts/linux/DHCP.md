---
title: DHCP
date: 2017-09-16 09:00:00
categories: linux
tags:
- DHCP
---

　　`DHCP`（`Dynamic Host Configuration Protocol`，`动态主机配置协议`）是一个局域网的网络协议，使用UDP协议工作， 主要有两个用途：给内部网络或网络服务供应商自动分配IP地址，给用户或者内部网络管理员作为对所有计算机作中央管理的手段。DHCP主要2个端口，其中`UDP67`为DHCP Server的服务端口，`UDP68`为DHCP Client的服务端口；


<!--more-->

---

## 简介

　　dhcp的前身是`bootp` ： BOOTP（Bootstrap Protocol，引导程序协议）是一种引导协议，基于IP/UDP协议，也称自举协议，是DHCP协议的前身。BOOTP用于无盘工作站的局域网中，可以让无盘工作站从一个中心服务器上获得IP地址。通过BOOTP协议可以为局域网中的无盘工作站分配动态IP地址，这样就不需要管理员去为每个用户去设置静态IP地址。




　　BOOTP使用UDP报文传输，并使用保留端口号`67（BOOTP服务器）`和`68（BOOTP客户端）`工作。使用BOOTP协议的时候，一般包括Bootstrap Protocol Server（自举协议服务端）和Bootstrap Protocol Client（自举协议客户端）两部分。

**BOOTP的一般工作流程就是BOOTP客户端和BOOTP服务器之间的交互，其流程如下：**

>- 由BOOTP启动代码来启动BOOTP客户端，这个时候BOOTP客户端还没有IP地址。
>- BOOTP客户端使用广播形式的IP地址255.255.255.255向网络中发出IP地址查询要求。
>- 运行BOOTP协议的服务器接收到这个请求，会根据请求中提供的MAC地址找到BOOTP客户端，并发送一个含有IP地址、服务器IP地址、网关等信息的回应帧。
>- BOOTP客户端会根据该回应帧来获得自己的IP地址并通过专用文件服务器（如TFTP服务器）下载启动镜像文件，模拟成磁盘来完成启动。

---

## DHCP由来

　　BOOTP实现了对于主机地址的静态配置，根据主机mac地址，分配一个静态ip。而DHCP实现了动态配置。

　　我们熟知的DHCP协议是从BOOTP的基础上发展而来的，它们都是主机配置协议，都可以大大减少管理员的工作量。BOOTP可以看成是简单版的DHCP，是对主机的静态配置，而DHCP可以依据一些策略对主机进行动态配置。BOOTP用于无盘工作站的启动和配置，而DHCP更适用于客户端接入变化的网络，即客户端接入时间、接入地点不固定。


　　在DHCP中为了实现动态配置引入了`租约`的概念，申请IP地址使用时长，时间到了续租或者重新分配。



## DHCP

`RARP`：MAC->IP        
`ARP`：IP->MAC 

---

### 八种报文

DHCP共有八种报文

>`DHCP DISCOVER`：客户端到服务器
`DHCP OFFER` ：服务器到客户端
`DHCP REQUEST`：客户端到服务器
`DHCP ACK` ：服务器到客户端
`DHCP NAK`：服务器到客户端,通知用户无法分配合适的IP地址
`DHCP DECLINE` ：客户端到服务器，指示地址已被使用
`DHCP RELEASE`：客户端到服务器，放弃网络地址和取消剩余的租约时间
`DHCP INFORM`：客户端到服务器,客户端如果需要从DHCP服务器端获取更为详细的配置信息，则发送Inform报文向服务器进行请求


### DHCP流程

<div align=center>
![](/images/linux/dhcp02.png)
</div>


>1、`dhcp discover` 在本网络内询问谁是dhcp服务器，广播方式
>2、`dhcp offer `：dhcp服务器 分配ip/netmask gateway给客户端
>3、`dhcp request` 请求使用报文，广播告诉使用哪一台dhcp server提供的ip，其他的dhcp server回收ip
>>多台dhcp服务器响应，先到先得，决定使用哪个
>>如果分配的ip已被使用

>4、`dhcp server ack` 确认，广播方式


---

### 续租

>client : `dhcp request` 单播
server : `dhcp ack`  继续租
server : `dhcp nak` 不继续给租了，就需要discover 广播重新找地址了



## 安装配置

　　下面，来看看DHCP如何安装配置

### 包列表

　　查看一下dhcp包里包含哪里文件

```shell
[ root@centos69 ~ ]# rpm -ql dhcp
/etc/dhcp
/etc/dhcp/dhcpd.conf                    # dhcp 配置文件
/etc/dhcp/dhcpd6.conf                   # ipv6 dhcp配置文件
/etc/openldap/schema/dhcp.schema
/etc/portreserve/dhcpd
/etc/rc.d/init.d/dhcpd                  # 脚本
/etc/rc.d/init.d/dhcpd6                 # IPV6 dhcp服务器服务脚本
/etc/rc.d/init.d/dhcrelay               # 中继器服务脚本
/etc/rc.d/init.d/dhcrelay6
/etc/sysconfig/dhcpd
/etc/sysconfig/dhcpd6
/etc/sysconfig/dhcrelay
/etc/sysconfig/dhcrelay6
/usr/bin/omshell
/usr/sbin/dhcpd                         # 服务器
/usr/sbin/dhcrelay                      # 中继器
/var/lib/dhcpd
/var/lib/dhcpd/dhcpd.leases
/var/lib/dhcpd/dhcpd6.leases
```



### 配置文件示例

　　配置 都要以 分号`";"`结尾 否则语法错误

　　subnet     地址池分配IP

　　filename "vmunix.passacaglia";         指明引导文件名称和路径

　　next-server     # server-name 指明主机地址  从哪台主机加载引导文件


```shell
[ root@centos69 ~ ]# cat /usr/share/doc/dhcp*/dhcpd.conf.sample
# 全局配置
default-lease-time 600;                           # 默认租约期限 单位秒
max-lease-time 7200;                              # 最大租约期限
option domain-name "example.org";                 # 搜索域，ping www 自动补全
# 可选项 可以定义在全局，也可以定义在subnet中，也可以放在Host中
# 作用范围越小的，优先级越高
option domain-name-servers ns1.example.org, ns2.example.org;    # 域名服务器
# option domain-name-servers 172.18.0.1 写成ip地址
# 地址池
# 当前主机所在网络，
subnet 10.254.239.32 netmask 255.255.255.224 {
  range dynamic-bootp 10.254.239.40 10.254.239.60;  # 地址池，起始地址，结束地址
  option broadcast-address 10.254.239.31;           # 广播地址
  option routers rtr-239-32-1.example.org;          # 网关，告诉客户端网关地址（IP地址）
}
# 保留地址（某些主机固定分配地址）
host passacaglia {
  hardware ethernet 0:0:c0:5d:bd:95;
  filename "vmunix.passacaglia";                    # 指明引导文件名称和路径
  server-name "toccata.fugue.com";
}
# 超级“作用域”（多个作用域，一个服务器分配）
shared-network 224-29 {
  subnet 10.17.224.0 netmask 255.255.255.0 {
    option routers rtr-224.example.org;
  }
  subnet 10.0.29.0 netmask 255.255.255.0 {
    option routers rtr-29.example.org;
  }
  pool {
    allow members of "foo";
    range 10.17.224.10 10.17.224.250;
  }
  pool {
    deny members of "foo";
    range 10.0.29.10 10.0.29.230;
  }
}
```




## dhclient

> dhclient - Dynamic Host Configuration Protocol Client

能启动，只能启动一次

dhclient -d  # 前台查看
  
dhclient 监听在 udp 68号端口

<div align=center>
![](/images/linux/dhcp01.png)
</div>



## 查看某个ip分配给谁了


> cat `/var/lib/dhcpd/dhcpd.leases`     * 租约文件，记录地址分配结果

```shell
[ imkindu@centos69x ~ ]$ cat /var/lib/dhcpd/dhcpd.leases
# The format of this file is documented in the dhcpd.leases(5) manual page.
# This lease file was written by isc-dhcp-4.1.1-P1

server-duid "\000\001\000\001!P z\000\014)\374\276\323";

lease 192.168.10.5 {                        # 分配的IP
  starts 6 2017/09/16 17:51:07;             # 开始时间
  ends 0 2017/09/17 05:51:07;               # 到期时间
  cltt 6 2017/09/16 17:51:07;
  binding state active;
  next binding state free;
  hardware ethernet 00:0c:29:fc:3f:56;      # Mac地址
}

lease 192.168.10.6 {
  starts 6 2017/09/16 18:06:13;
  ends 0 2017/09/17 06:06:13;
  cltt 6 2017/09/16 18:06:13;
  binding state active;
  next binding state free;
  hardware ethernet 00:0c:29:fc:3f:56;
}
```


---

## 检查语法

> service dhcpd configtest

```shell
[ root@centos69x CentOS7 ]# service dhcpd configtest
Syntax: OK
```

---

## dnsmasq

```shell
[ root@centos69 ~ ]# yum info dnsmasq
Loaded plugins: fastestmirror, security
Loading mirror speeds from cached hostfile
Installed Packages
Name        : dnsmasq
Arch        : x86_64
Version     : 2.48
Release     : 17.el6
Size        : 293 k
Repo        : installed
From repo   : media
Summary     : A lightweight DHCP/caching DNS server
URL         : http://www.thekelleys.org.uk/dnsmasq/
License     : GPLv2 or GPLv3
Description : Dnsmasq is lightweight, easy to configure DNS forwarder and DHCP server.
            : It is designed to provide DNS and, optionally, DHCP, to a small network.
            : It can serve the names of local machines which are not in the global
            : DNS. The DHCP server integrates with the DNS server and allows machines
            : with DHCP-allocated addresses to appear in the DNS with names configured
            : either in each host or in a central configuration file. Dnsmasq supports
            : static and dynamic DHCP leases and BOOTP for network booting of diskless
            : machines.
```

