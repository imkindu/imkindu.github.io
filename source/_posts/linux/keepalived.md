---
title: keepalived
date: 2017-10-30 21:00:00
author: imkindu
categories: linux
tags:
- keepalived
---


　　keepalived是集群管理中保证集群高可用的一个服务软件，其功能类似于heartbeat，用来防止单点故障。

<!--more-->


## 简介


　　Keepalived是Linux下一个轻量级的高可用解决方案，它与HeartBeat、RoseHA实现的功能类似，都可以实现服务或者网络的高可用，但是又有差别：HeartBeat是一个专业的、功能完善的高可用软件，它提供了HA软件所需的基本功能，比如心跳检测和资源接管，监测集群中的系统服务，在群集节点间转移共享IP地址的所有者等，HeartBeat功能强大，但是部署和使用相对比较麻烦；与HeartBeat相比，Keepalived主要是通过虚拟路由冗余来实现高可用功能，虽然它没有HeartBeat功能强大，但Keepalived部署和使用非常简单，所有配置只需一个配置文件即可完成。




## 工作原理



　　keepalived是以`VRRP`协议为实现基础的，VRRP全称`Virtual Router Redundancy Protocol`，即虚拟路由冗余协议。

　　虚拟路由冗余协议，可以认为是实现路由器高可用的协议，即将N台提供相同功能的路由器组成一个路由器组，这个组里面有一个master和多个backup，master上面有一个对外提供服务的vip（该路由器所在局域网内其他机器的默认路由为该vip），master会发组播，当backup收不到vrrp包时就认为master宕掉了，这时就需要根据VRRP的优先级来选举一个backup当master。这样的话就可以保证路由器的高可用了。

　　keepalived主要有三个模块，分别是core、check和vrrp。core模块为keepalived的核心，负责主进程的启动、维护以及全局配置文件的加载和解析。check负责健康检查，包括常见的各种检查方式。vrrp模块是来实现VRRP协议的。


<div align="center">
![](/images/keepalived/01.png)
</div>


- `WatchDog` ：负载监控checkers和VRRP进程的状况
- `VRRP Stack` ：负载负载均衡器之间的失败切换FailOver，如果只用一个负载均稀器，则VRRP不是必须的。
- `Checkers` ：负责真实服务器的健康检查healthchecking，是keepalived最主要的功能。
- `IPVS wrapper` ：用户发送设定的规则到内核ipvs代码
- `Netlink Reflector` ：用来设定vrrp的vip地址等。




## keepalived配置前提

HA Cluster配置前提

```text
1、本机主机名与hosts中定义的主机名保持一致，要与hostname(uname -n)获得的名称保持一致
	centos6 : /etc/sysconfig/network
	centos7 : hostnamectl set-hostname HOSTNAME

	各节点要能互相解析主机名：一般通过Hosts文件进行解析，不依赖于外置网络和DNS
2、各节点时间同步
3、确保iptables和selinux关闭
4、各节点之间的root用户可以基于密钥认证的ssh服务完成互相通信（对KA并非必须）
```


## VRRP

　　VRRP将局域网内的一组路由器划分在一起，形成一个VRRP备份组，它在功能上相当于一台虚拟路由器，使用虚拟路由器号进行标识。以下使用虚拟路由器代替VRRP备份组进行描述。

　　虚拟路由器有自己的虚拟IP地址和虚拟MAC地址，它的外在表现形式和实际的物理路由器完全一样。局域网内的主机将虚拟路由器的IP地址设置为默认网关，通过虚拟路由器与外部网络进行通信。

　　虚拟路由器是工作在实际的物理路由器之上的。它由多个实际的路由器组成，包括一个Master路由器和多个Backup路由器。Master路由器正常工作时，局域网内的主机通过Master与外界通信。当Master路由器出现故障时，Backup路由器中的一台设备将成为新的Master路由器，接替转发报文的工作。

<div align="center">
![](/images/keepalived/02.png)
</div>



### 相关术语

> `虚拟路由器`：由一个 Master 路由器和多个 Backup 路由器组成。主机将虚拟路由器当作默认网关。
`VRID`：虚拟路由器的标识。有相同 VRID 的一组路由器构成一个虚拟路由器。
`Master 路由器`：虚拟路由器中承担报文转发任务的路由器。
`Backup 路由器`：Master 路由器出现故障时，能够代替 Master 路由器工作的路由器。
`虚拟 IP 地址`：虚拟路由器的 IP 地址。一个虚拟路由器可以拥有一个或多个IP 地址。
`IP 地址拥有者`：接口 IP 地址与虚拟 IP 地址相同的路由器被称为 IP 地址拥有者。
`虚拟 MAC 地址`：一个虚拟路由器拥有一个虚拟 MAC 地址。虚拟 MAC 地址的格式为 00-00-5E-00-01-{VRID}。通常情况下，虚拟路由器回应 ARP 请求使用的是虚拟 MAC 地址，只有虚拟路由器做特殊配置的时候，才回应接口的真实 MAC 地址。
`优先级`：VRRP 根据优先级来确定虚拟路由器中每台路由器的地位。
`非抢占方式`：如果 Backup 路由器工作在非抢占方式下，则只要 Master 路由器没有出现故障，Backup 路由器即使随后被配置了更高的优先级也不会成为Master 路由器。
`抢占方式`：如果 Backup 路由器工作在抢占方式下，当它收到 VRRP 报文后，会将自己的优先级与通告报文中的优先级进行比较。如果自己的优先级比当前的 Master 路由器的优先级高，就会主动抢占成为 Master 路由器；否则，将保持 Backup 状态。


### 广播地址

　　master以广播的方式通知其他 自己还在工作，为了保证master的广播是实时通知给其他slave，中间没有延迟，必须保证`两边时间一致`

　　广播地址(Broadcast Address)是专门用于同时向网络中所有工作站进行发送的一个地址。在使用TCP/IP 协议的网络中，主机标识段host ID 为全1 的IP 地址为广播地址，广播的分组传送给host ID段所涉及的所有计算机。


### 时间同步

ntp : network time protocol


linux服务器时间差别为什么很大？需要经常同步


ntp调整时间，不是直接将时间调整到时间服务器当前时间，如果这样的话，当前服务器会跳过中间一段时间，没有记录，这是不合适的，ntp的做法是：加快当前服务器时间的走速，尽快跟上时间服务器的时间

``` shell
[root@centos49 keepalived]# ntpdate 172.18.0.1  # 直接调整时间，跳过中间间隔时间
```







## keepalived


　　keepalived主要使用了VRRP协议，基本配置参数同VRRP。下面一起安装操作一遍。


### 安装
	
```shell	
[root@centos49 keepalived]# yum install keepalived

[root@centos49 keepalived]# yum info keepalived
Loaded plugins: fastestmirror, security
Loading mirror speeds from cached hostfile
Installed Packages
Name        : keepalived
Arch        : x86_64
Version     : 1.2.13
Release     : 5.el6_6
Size        : 625 k
Repo        : installed
From repo   : base
Summary     : Load balancer and high availability service
URL         : http://www.keepalived.org/
License     : GPLv2+
Description : Keepalived provides simple and robust facilities for load balancing
            : and high availability.  The load balancing framework relies on the
            : well-known and widely used Linux Virtual Server (IPVS) kernel module
            : providing layer-4 (transport layer) load balancing.  Keepalived
            : implements a set of checkers to dynamically and adaptively maintain
            : and manage a load balanced server pool according their health.
            : Keepalived also implements the Virtual Router Redundancy Protocol
            : (VRRPv2) to achieve high availability with director failover.

```

## keepalived配置

　　keepalived主要配置文件`/etc/keepalived/keepalived.conf`，可以通过`man keepalived.conf`查看说明

　　keepalived配置文件主要分为三部分

```shell
GLOBAL CONFIGURATION				# 全局配置
	Global definitions
	Static routes/addresses
VRRPD CONFIGURATION 			    # VRRP路由实例配置
	VRRP synchronization group(s)：vrrp同步组
	VRRP instance(s)：即一个vrrp虚拟路由器
LVS CONFIGURATION 					# lvs集群配置
	Virtual server group(s)
	Virtual server(s)：ipvs集群的vs和rs
```



### 全局配置

```shell
global_defs           		# Block id
    {
    notification_email    	# To: 异常发送至指定管理员邮箱，可以多个
           {
           admin@example1.com
           ...
           }
   	
    notification_email_from admin@example.com   # From: from address that will be in header 发送人
    smtp_server 127.0.0.1        	# IP 邮件服务器地址
    smtp_connect_timeout 30      	# integer, seconds 超时时间
    router_id my_hostname           # string identifying the machine, 虚拟路由Id
                                 	# (doesn’t have to be hostname).
    vrrp_mcast_group4 224.0.0.18 	# optional, default 224.0.0.18 广播地址，可以不用配置，多个虚拟路由使用同一个广播地址容易冲突
    vrrp_mcast_group6 ff02::12   	# optional, default ff02::12
    enable_traps                 	# enable SNMP traps 
}
```


### 实例配置


```text
vrrp_instance VI_1 {
    state MASTER 			# 状态 MASTER|BACKUP
    interface eth0 			# 虚拟IP绑定在哪块网卡：interface for inside_network, bound by vrrp
    virtual_router_id 51 	# 虚拟路由器ID
    priority 100 			# 优先级0-255，数字越大，优先级越高
    advert_int 1 			# 主节点 每隔多次时间 发送心跳 1s
    authentication { 		# VRRP 认证方式：无认证、简单字符认证、MD5认证
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.200.16 		# 虚拟IP地址
        192.168.200.17		# <IPADDR>/<MASK> brd <IPADDR> dev <STRING> scope <SCOPE> label <LABEL>
        192.168.200.18 		# 192.168.200.18/24 dev eth2 label eth2:1
    }
    # nopreempt 			# 非抢占模式，默认抢占模式

    定义通知脚本：
	notify_master <STRING>|<QUOTED-STRING>：	# 当前节点成为主节点时触发的脚本 notify_master"/etc/keepalived/notify.sh master"
	notify_backup <STRING>|<QUOTED-STRING>：	# 当前节点转为备节点时触发的脚本 notify_backup"/etc/keepalived/notify.sh backup"
	notify_fault <STRING>|<QUOTED-STRING>：	# 当前节点转为“失败”状态时触发的脚本 notify_fault"/etc/keepalived/notify.sh fault"
	notify <STRING>|<QUOTED-STRING>：		# 通用格式的通知触发机制，一个脚本可完成以上三种状态的转换时的通知
}
```

### vrrp示例

　　利用keepalived的vrrp_instance实现ip地址高可用性

```shell
vrrp_instance VI_20 {
    state MASTER
    interface eth1
    virtual_router_id 20
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass imkindu
    }
    virtual_ipaddress {
        192.168.56.20/16 dev eth1 label eth1:1 
    }
}
```

　　master会通过广播地址想其他backup多播，声明自己的权重 和 正在运行，如果backup接受不到master的广播，就会接管vip，并开始广播

```
[ root@centos50 keepalived ]# tcpdump -i eth1 -nn vrrp
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on eth1, link-type EN10MB (Ethernet), capture size 65535 bytes
20:44:21.895151 IP 192.168.56.49 > 224.0.0.18: VRRPv2, Advertisement, vrid 20, prio 100, authtype simple, intvl 1s, length 20
20:44:22.896702 IP 192.168.56.49 > 224.0.0.18: VRRPv2, Advertisement, vrid 20, prio 100, authtype simple, intvl 1s, length 20
```


### 通知脚本

```shell
#!/bin/bash
#
contact='root@localhost'
notify() {
	mailsubject="$(hostname) to be $1, vip floating"
	mailbody="$(date +'%F %T'): vrrp transition, $(hostname) changed to be $1"
	echo "$mailbody" | mail -s "$mailsubject" $contact
}
case $1 in
	master)
		notify master
		;;
	backup)
		notify backup
		;;
	fault)
		notify fault
		;;
	*)
		echo "Usage: $(basename $0) {master|backup|fault}"
		exit 1
		;;
esac
```



### 同步组

　　lvs+keepalived里面  如果Lvs 使用NAT模式，real server 需要指向director的DIP
　　如果配置了keepalived，VIP迁移了，dip也需要迁移，这里就需要使用到 同步组


virtual_server fwmark int 一起调度


```shell
	vrrp_sync_group VG_1 {
		group {
			VI_1 				# vip和dip一起漂移
			VI_2
		}
	}

	vrrp_instance VI_1 {
		eth0
		vip
	}

	vrrp_instance VI_2 {
		eth1
		dip
	}
```

多个虚拟路由，多播地址不能全局指定，都用一个多播地址，就混乱了



## 脚本控制优先级

　　keepalived调用外部的辅助脚本进行资源监控，并根据监控的结果状态能实现优先动态调整

　　vrrp_script:自定义资源监控脚本，vrrp实例根据脚本返回值，公共定义，可被多个实例调用，**定义在vrrp实例之外**

```shell
	vrrp_script chk_down {
		script "[[ -f /etc/keepalived/down ]] && exit 1 || exit 0" 		# 定义脚本，如果down文件存在，退出1，并权重减2
		interval 1 														# 间隔时间
		weight -20 														# 如何失败执行，权重减20
	}
```

　　track_script:调用vrrp_script定义的脚本去监控资源，定义在实例之内，调用事先定义的vrrp_script

```shell
	track_script {
		chk_down
	}
```

---

## keepalived 日志


```shell
[ root@centos50 keepalived ]# cat /etc/sysconfig/keepalived 
# Options for keepalived. See `keepalived --help' output and keepalived(8) and
# keepalived.conf(5) man pages for a list of all options. Here are the most
# common ones :
#
# --vrrp               -P    Only run with VRRP subsystem.
# --check              -C    Only run with Health-checker subsystem.
# --dont-release-vrrp  -V    Dont remove VRRP VIPs & VROUTEs on daemon stop.
# --dont-release-ipvs  -I    Dont remove IPVS topology on daemon stop.
# --dump-conf          -d    Dump the configuration data.
# --log-detail         -D    Detailed log messages.
# --log-facility       -S    0-7 Set local syslog facility (default=LOG_DAEMON)
#

KEEPALIVED_OPTIONS="-D"
```

### 指定keepalived日志

通过修改keepalived的参数选项 `-S` 自定义日志

```shell
KEEPALIVED_OPTIONS="-D -S 3"
```

### 修改rsyslog配置

```shell
[ root@centos50 keepalived ]# vim /etc/rsyslog.conf
local3.*                                                /var/log/keepalived.log
```

### 查看日志

```shell
[ root@centos50 keepalived ]# tail -f /var/log/keepalived.log 
Oct 30 21:17:00 centos50 Keepalived_vrrp[24305]: VRRP_Instance(VI_20) Entering BACKUP STATE
Oct 30 21:17:00 centos50 Keepalived_healthcheckers[24304]: Configuration is using : 7679 Bytes
Oct 30 21:17:00 centos50 Keepalived_healthcheckers[24304]: Using LinkWatch kernel netlink reflector...
Oct 30 21:17:00 centos50 Keepalived_vrrp[24305]: VRRP sockpool: [ifindex(2), proto(112), unicast(0), fd(10,11)]
Oct 30 21:17:01 centos50 Keepalived_vrrp[24305]: VRRP_Instance(VI_21) Transition to MASTER STATE
Oct 30 21:17:02 centos50 Keepalived_vrrp[24305]: VRRP_Instance(VI_21) Entering MASTER STATE
Oct 30 21:17:02 centos50 Keepalived_vrrp[24305]: VRRP_Instance(VI_21) setting protocol VIPs.
Oct 30 21:17:02 centos50 Keepalived_vrrp[24305]: VRRP_Instance(VI_21) Sending gratuitous ARPs on eth0 for 172.18.56.21
Oct 30 21:17:02 centos50 Keepalived_healthcheckers[24304]: Netlink reflector reports IP 172.18.56.21 added
Oct 30 21:17:07 centos50 Keepalived_vrrp[24305]: VRRP_Instance(VI_21) Sending gratuitous ARPs on eth0 for 172.18.56.21
```


---



## KeepAlived支持IPVS


```shell
	man keepalived.conf


	Virtual server(s)
       A virtual_server can be a declaration of one of


	delay_loop <INT>： 						检查后端服务器的时间间隔
	lb_algorr|wrr|lc|wlc|lblc|sh|dh：		定义调度方法
	lb_kind NAT|DR|TUN：					集群的类型
	persistence_timeout <INT>： 			持久连接时长
	protocol TCP： 							服务协议，仅支持TCP
	sorry_server<IPADDR> <PORT>： 			所有RS故障时，备用服务器地址
	virtualhost <STRING> 					定义虚拟主机
	real_server<IPADDR> <PORT>
	{
		weight <INT> RS权重
		notify_up<STRING>|<QUOTED-STRING> RS上线通知脚本
		notify_down<STRING>|<QUOTED-STRING> RS下线通知脚本
		HTTP_GET|SSL_GET|TCP_CHECK|SMTP_CHECK|MISC_CHEC K { ... }：定义当前主机的健康状态检测方法
	}


 	HTTP_GET|SSL_GET 											# real server 健康状态检测机制
    {
         # A url to test
         # can have multiple entries here
         url {
           #eg path / , or path /mrtg2/
           path <STRING> 										# 检测路径
           # healthcheck needs status_code
           # or status_code and digest
           # Digest computed with genhash
           # eg digest 9b3a0c85a887a256d6939da88aabd8cd
           digest <STRING> 										# 检测校验码
           # status code returned in the HTTP header
           # eg status_code 200
           status_code <INT> 									# 返回状态
         }
         # number of get retry
         nb_get_retry <INT>
         # delay before retry
         delay_before_retry <INT> 								# 每次重试之前间隔时间

         # ======== generic connection options
         # Optional IP address to connect to.
         # The default is real server’s IP
         connect_ip <IP ADDRESS> 								# 指明IP地址
         # Optional port to connect to if not
         # The default is real server’s port
         connect_port <PORT>
         # Optional interface to use to
         # originate the connection
         bindto <IP ADDRESS> 									# 从本机哪个地址发送检测
         # Optional source port to
         # originate the connection from
         bind_port <PORT>
         # Optional connection timeout in seconds.
         # The default is 5 seconds
         connect_timeout <INTEGER> 								# 连接超时时间
         # Optional fwmark to mark all outgoing
         # checker pakets with
         fwmark <INTEGER>

         # Optional random delay to begin initial check for
         # maximum N seconds.
         # Useful to scatter multiple simultaneous
         # checks to the same RS. Enabled by default, with
         # the maximum at delay_loop. Specify 0 to disable
         warmup <INT>
    } #HTTP_GET|SSL_GET

```












