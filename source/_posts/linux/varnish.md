---
title: varnish
date: 2017-10-31 09:00:00
author: imkindu
categories: linux
tags:
- varnish
---

　　nginx本身也具有缓存功能，但是缓存功能不是它的强项，真正的缓存另有其人：`varnish`,varnish 是squid的升级版，主要应用于http得反向代理和http缓存来提供加速功能 

<!--more-->

## 简介


　　Varnish是一款高性能的开源HTTP加速器，一般用来和Nginx、Apache等搭配使用，组建一个高效率的Web服务器。Varnish的某个负责接受新HTTP连接的线程开始等待用户，如果有新的HTTP连接过来，它总负责接收，然后叫醒某个等待中的线程。

　　Worker线程读入HTTP请求的URI，查找已有的object，如果命中则直接返回并回复用户。如果没有命中，则需要将所请求的内容，从后端服务器中取过来，存到缓存中，然后再回复。Varnish根据所读到object的大小，创建相应大小的缓存文件。


### 缓存概念

　　程序具有局部性

> `时间局部性`：过去访问的的数据，过一段时间可能再次访问到
>
> `空间局部性`：一个数据被访问到，离它较近的数据也可能访问到



命中 ： 获取到缓存数据

命中率:hit/(hit+miss)

> `文档命中率` : 从文档个数进行衡量
> 
> `字节命中率` : 从内容大小进行衡量 



缓存对象 ： 生命周期  定期清理

缓存空间耗尽 ： LRU （最近最少使用原则）


缓存处理步骤

> 接收请求 -> 解析请求 -> 查询缓存 -> 新鲜度检测 -> 创建响应报文 -> 发送响应 -> 写入日志




### 缓存分类


> 代理式缓存
>>	squid
>>	varnish

> 旁挂式缓存




### 新鲜度检测机制
　　


**过期时间检测**

- `HTTP/1.0 Expires` : expires:Mon, 06 Nov 2017 12:28:49 GMT
- `HTTP/1.1 Cache-Control` : max-age=600

**有效性再验证**

1、文件修改时间是否发送改变
2、文件内容`etag` 标签校验

- 如果原始内容未改变，则仅响应首部（不附带body部分）,响应码304（Not Modified）
- 如果原始内容发生改变，则正常响应，响应码200
- 如果原始内容消失，则响应404，此时缓存中的cache object也应该被删除

**条件式请求首部**

- `If-Modified-Sine` *自从某个时间是否发送改变
- `If-Unmodified-Sine` 自从某个时间是否为发生改变
- `If-Math` 是否匹配
- `If-None-Match` *是否不匹配:


```shell
Cache-Control   = "Cache-Control" ":" 1#cache-directive
	cache-directive = cache-request-directive
	     | cache-response-directive

	请求报文用于通知缓存服务器如何使用缓存响应请求
		cache-request-directive =
		       "no-cache"  									(不要缓存的实体，要求现在从WEB服务器去取)                        
		     | "no-store" (backup)                          
		     | "max-age" "=" delta-seconds         
		     | "max-stale" [ "=" delta-seconds ]  			（可以接受过去的对象，但是过期时间必须小于 max-stale 值） 
		     | "min-fresh" "=" delta-seconds      
		     | "no-transform"                      
		     | "only-if-cached"                   
		     | cache-extension  

	 响应报文用于通知缓存服务器如何存储上级服务器响应的内容                 
		 cache-response-directive =
		       "public"                               
		     | "private" [ "=" <"> 1#field-name <"> ] 
		     | "no-cache" [ "=" <"> 1#field-name <"> ]
		     | "no-store"                            
		     | "no-transform"                         
		     | "must-revalidate"                     
		     | "proxy-revalidate"                    
		     | "max-age" "=" delta-seconds            		公共缓存+私有缓存
		     | "s-maxage" "=" delta-seconds  				公共缓存         
		     | cache-extension 
```


## 常见缓存服务

常见的缓存服务开源解决方案

- `varnish`
- `squid`


类似于nginx 和 apache的关系

---


## varnish结构

<div align="center">
![](/images/varnish/001.png)
</div>

Varnish主要运行两个进程：`Management`进程和`Child`进程(也叫Cache进程)。


> `management` : 编译VCL并应用新配置、监控varnish、初始化varnish、CLI接口
>> Command line 命令行管理工具
>> Child process mgmt 子进程管理
>> initialisation 初始化 
>
> `child/cache` 
>> Commad line 线程 : 管理接口
>> Storage/hashing 线程 ：缓存存储
>> Log/stats 线程：日志管理线程
>> Backend Communication 线程：管理后端主机线程
>> Accept : 接受新的连接请求
>> worker threads : 处理用户请求，child进程会为每个会话启动一个worker线程
>> Object Expiry : 清理缓存中的过期对象
>`vcl` : varnish configuration language，基于"域" 的编程语言，花括号{}括起来，


## varnish 安装

　　在centos6 varnish还是2.1.5版本，过于老了，所有需要自己编译安装，在centos7上光盘自带varnish4版本，可以直接通过yum安装

``` shell
[root@centos72 ~]# yum search varnish
Loaded plugins: fastestmirror, langpacks
Loading mirror speeds from cached hostfile
==== N/S matched: varnish ======
collectd-varnish.x86_64 : Varnish plugin for collectd
varnish-docs.x86_64 : Documentation files for varnish
varnish-libs.x86_64 : Libraries for varnish
varnish-libs-devel.x86_64 : Development files for varnish-libs
varnish.x86_64 : High-performance HTTP accelerator
```


## varnish程序环境

`/etc/varnish/varnish.params` ： 配置varnish服务进程的工作特性，例如监听的地址和端口，缓存机制；
`/etc/varnish/default.vcl` ：配置各Child/Cache线程的缓存策略；

>主程序：
　　/usr/sbin/varnishd
CLI interface：
　　/usr/bin/varnishadm
Shared Memory Log交互工具：
　　/usr/bin/varnishhist
　　/usr/bin/varnishlog
　　/usr/bin/varnishncsa
　　/usr/bin/varnishstat
　　/usr/bin/varnishtop		
测试工具程序：
　　/usr/bin/varnishtest
VCL配置文件重载程序：
　　/usr/sbin/varnish_reload_vcl
Systemd Unit File：
　　/usr/lib/systemd/system/varnish.service　          #varnish服务
　　/usr/lib/systemd/system/varnishlog.service　   # 日志持久的服务
　　/usr/lib/systemd/system/varnishncsa.service	
        


## varnish 存储

　　Varnish支持多种不同类型的后端存储，这可以在varnishd启动时使用-s选项指定。后端存储的类型包括：

- `file` : 单个文件缓存，不支持持久机制，Key值缓存在内存，重启丢失,所有数据放在一个黑盒中（基于文件系统上单独的一个文件系统）
- `malloc` : 全部缓存在内存中 jemalloc
- `persistent` : 基于file文件的形式持久缓存,但可以持久存储数据(即重启varnish数据时不会被清除)；仍处于测试期；




## varnish配置




### 配置varnish的三种应用

1、varnishd应用程序的命令行参数

	监听的socket，使用的存储类型等等
	-p param=value
	-r param,param... 设定只读参数列表

	/etc/varnish/varnish.params

2、-p 选项指定的参数

	运行时参数：可在程序运行中，通过其CLI进行配置

3、vcl

	配置缓存系统的缓存机制
	通过vcl配置文件进行配置，先编译，后应用，依赖于c编译器




## varnish日志

　　shared memory log : 共享内存日志大小一般默认为90MB，分为两部分，前一部分为计数器，后一部分为请求相关数据。

　　为了与系统的其它部分进行交互，Child进程使用了可以通过文件系统接口进行访问的共享内存日志(shared memory log)，因此，如果某线程需要记录信息，其仅需要持有一个锁，而后向共享内存中的某内存区域写入数据，再释放持有的锁即可。而为了减少竞争，每个worker线程都使用了日志数据缓存。共享内存日志大小一般为90M，其分为两部分，前一部分为计数器，后半部分为客户端请求的数据。varnish提供了多个不同的工具如varnishlog、varnishncsa或varnishstat等来分析共享内存日志中的信息并能够以指定的方式进行显示。

统计数据：计数器
日志区域：日志记录


```shell
/usr/bin/varnishhist
/usr/bin/varnishlog
/usr/bin/varnishncsa
/usr/bin/varnishstat
/usr/bin/varnishtest
/usr/bin/varnishtop
```

　　varnish日志有一个单独的服务`varnishlog`，启动该服务，可以将日志写入日志文件。查看服务脚本可以看到，默认日志文件是`/var/log/varnish/varnish.log`。

``` shell
[root@centos72 ~]# systemctl status varnishlog.service 
● varnishlog.service - Varnish Cache HTTP accelerator logging daemon
   Loaded: loaded (/usr/lib/systemd/system/varnishlog.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
[root@centos72 ~]# cat /usr/lib/systemd/system/varnishlog.service 
[Unit]
Description=Varnish Cache HTTP accelerator logging daemon
After=varnish.service

[Service]
RuntimeDirectory=varnishlog
Type=forking
PIDFile=/run/varnishlog/varnishlog.pid
User=varnish
Group=varnish
ExecStart=/usr/bin/varnishlog -a -w /var/log/varnish/varnish.log -D -P /run/varnishlog/varnishlog.pid
ExecReload=/bin/kill -HUP $MAINPID

[Install]
WantedBy=multi-user.target
```



## vcl

　　Varnish Configuration Language (VCL)是varnish配置缓存策略的工具，它是一种基于“域”(domain specific)的简单编程语言，它支持有限的算术运算和逻辑运算操作、允许使用正则表达式进行字符串匹配、允许用户使用set自定义变量、支持if判断语句，也有内置的函数和变量等。使用VCL编写的缓存策略通常保存至.vcl文件中，其需要编译成二进制的格式后才能由varnish调用。事实上，整个缓存策略就是由几个特定的子例程如vcl_recv、vcl_fetch等组成，它们分别在不同的位置(或时间)执行，如果没有事先为某个位置自定义子例程，varnish将会执行默认的定义。

　　VCL策略在启用前，会由management进程将其转换为C代码，而后再由gcc编译器将C代码编译成二进制程序。编译完成后，management负责将其连接至varnish实例，即child进程。正是由于编译工作在child进程之外完成，它避免了装载错误格式VCL的风险。因此，varnish修改配置的开销非常小，其可以同时保有几份尚在引用的旧版本配置，也能够让新的配置即刻生效。编译后的旧版本配置通常在varnish重启时才会被丢弃，如果需要手动清理，则可以使用varnishadm的vcl.discard命令完成。


### vcl状态引擎

　　在varnish里面同iptables里面的四表五链一样，可以配置什么文件缓存，什么文件不缓存等等，在vcl里面这里的钩子我们称为`状态引擎`


<div align="center">
![](/images/varnish/002.png)
</div>



### VCL函数


|   函数   |   作用   |      
| ---- | ---- |
|  vcl_recv    |   用户请求成功接收后，遇到的第一个函数，可以在这里对请求的数据进行处理，并决定选取下一步的处理策略   |      
| vcl_fetch |	从后端主机获取内容，并判断是否缓冲此内容，然后发送给客户端 |
| vcl_hash	| 对URL进行hash，可以自定义hash键 |
| vcl_pass	| 将请求直接发给backend，而不是用缓存中的数据 |
| vcl_hit |	在缓存中找到缓存对象时，要执行的操作|
| vcl_miss	| 在缓存中未找到对象时，要执行的操作|
| vcl_deliver |	响应给客户端时调用此方法 |
| vcl_pipe	| 不经由varnish直接将请求发往后端主机的时候调用，请求和内容不做任何改变，如同为客户端和backend建立一个管道 |
| vcl_error |	在varnishi上合成错误响应页时，调用此函数 |
|vcl_backend_fetch |向后端主机发送请求前，调用此函数，可修改发往后端的请求；|
|vcl_backend_response | 获得后端主机的响应后，可调用此函数；|
|vcl_backend_error | 当从后端主机获取源文件失败时，调用此函数；|
|vcl_init | VCL加载时调用此函数，经常用于初始化varnish模块(VMODs)|
|vcl_fini |当所有请求都离开当前VCL，且当前VCL被弃用时，调用此函数，经常用于清理varnish模块；|


这些函数类似或就是回调函数，是Vanish调用用户逻辑的接口。




### 内置变量


<div align="center">
![](/images/varnish/10.png)
</div>


- req：The request object，请求到达时可用的变量

>req.url     
req.http     
req.http.header     
req.restart     
server.ip     
server.hostname     
server.port     
req.backend

- bereq：The backend request object，向后端主机请求时可用的变量

>bereq.url     
bereq.http     
bereq.http.header     
bereq.proto     
bereq.connect_timeout

- obj：存储在内存中时对象属性相关的可用的变量

>obj.response     
obj.status     
obj.http.header     
obj.proto     
obj.ttl     
obj.hits

- beresp：The backend response object，从后端主机获取内容时可用的变量

>beresp.response     
beresp.http.header     
beresp.ttl     
beresp.proto     
beresp.do_gzip     
beresp.do_gunzip     
beresp.backend.name     
beresp.backend.ip

- resp：The HTTP response object，对客户端响应时可用的变量

>resp.response     
resp.proto     
resp.status     
resp.http.header

<div align="center">
![](/images/varnish/003.png)
</div>



### vcl语法


>(1) VCL files start with vcl 4.0;
(2) //, # and /* foo */ for comments;       # 注释
(3) Subroutines are declared with the sub keyword; 例如sub vcl_recv { ...}；
(4) No loops, state-limited variables（受限于引擎的内建变量）；
(5) Terminating statements with a keyword for next action as argument of the return() function, i.e.: return(action)；用于实现状态引擎转换； 
(6) Domain-specific;
			
The VCL Finite State Machine

>(1) Each request is processed separately;
(2) Each request is independent from others at any given time;
(3) States are related, but isolated;
(4) return(action); exits one state and instructs Varnish to proceed to the next state;
(5) Built-in VCL code is always present and appended below your own VCL;






## backend后端服务器

　　varnish是做缓存的，如果缓存服务器获取不到数据，需要去后台真实数据服务器获取，varnish可以配置后台服务器，也可以配置后端集群。

### 原始数据服务器

　　指明后端主机backend，修改default.vcl配置文件，指明backend，

```
backend default { 			# 为后台backend起一个名称
	.host = "172.18.56.52"; 	# 设定后台backend主机地址
	.port = "80";			# 设定后台backend主机端口
}
```

### 动静分离

　　配置多个后台backend，根据请求文件分发到不同服务器。

```
	backend default {
		.host = "172.18.56.50";
		.port = "80";
	}

	backend webimg {
		.host = "172.18.56.51";
		.port = "80";
	}

	# 设定两个backend后端主机，如果是php文件指定到一台服务器，如果是其他文件指定到默认服务器
	sub vcl_recv {
		if (req.url ~ "(?i)\.php$"){
			set req.backend_hint = webimg;
		}else{
			set req.backend_hint = default;
		}
	}
```

### varnish负载均衡


　　使用前需要导入 import directors;

　　varnish有两个特殊的引擎：

> `vcl_init`：在处理任何请求之前要执行的vcl代码：主要用于初始化VMODs；
`vcl_fini` ：所有的请求都已经结束，在vcl配置被丢弃时调用；主要用于清理VMODs；

``` shell
    import directors
    backend server1 {
        .host = "172.18.56.51";
        .port = "80";
    }
    backend server2 {
        .host = "172.18.56.52";
        .port = "80";
    }
	sub vcl_init{
		new GROUP_NAME = directors.round_robin();
		GROUP_NAME.add_backend(server1);
		GROUP_NAME.add_backend(server2);
	}
	sub vcl_recv {
		set req.backend.hint = GROUP_NAME.backend();	
	}
```


**deliver添加头部判断是否缓存**

　　在deliver添加一个Http头部，标识获取的资源是缓存的还是从backend获取的

```
sub vcl_deliver {
    # Happens when we have all the pieces we need, and are about to send the
    # response to the client.
    #
    # You can do accounting or modifying the final object here.
	if(obj.hits > 0) {
		set resp.http.X-cache = "Hit via " + server.ip; 					# 如果命中次数大于0 表示是缓存过的
	}else{
		set resp.http.X-cache = "Miss from " + server.ip;
	}
}
```
<div align="center">
![](/images/varnish/04.png)
</div>



基于cookie的session sticky

	init{
		new h = directors.hash();
		h.add_backend(one, 1); 			# 添加权重 1
	}

	sub vcl_recv {
		set req.backend_hint = h.backend(req.http.cookie);
	}
	



### 后端服务器状态监测


``` shell
# BE Health Check
backend BE_NAME {
    .host = 
    .port =
    .probe = {
        .url = 
        .timeout =
        .interval=
        .window=
        .threshold=
    }
}

.url 监测URL 默认为"/"
.interval 监测间隔时间
.window 最近多少次监测
.threshold 最近多少次成功算成功
```




















## varnish.params

　　varnish.params是一个重要的文件，设置varnish的环境变量，如：监听端口、配置文件、缓存位置、缓存大小、用户等。

``` python
	[root@centos73 development]# cat /etc/varnish/varnish.params 
	# Varnish environment configuration description. This was derived from
	# the old style sysconfig/defaults settings
	# Set this to 1 to make systemd reload try to switch VCL without restart.
	RELOAD_VCL=1
	# Main configuration file. You probably want to change it.
	VARNISH_VCL_CONF=/etc/varnish/default.vcl   # vcl配置文件
	# Default address and port to bind to. Blank address means all IPv4
	# and IPv6 interfaces, otherwise specify a host name, an IPv4 dotted
	# quad, or an IPv6 address in brackets.
	# VARNISH_LISTEN_ADDRESS=192.168.1.5
	VARNISH_LISTEN_PORT=6081                    # 监听端口
	# Admin interface listen address and port
	VARNISH_ADMIN_LISTEN_ADDRESS=127.0.0.1      # 监听端口
	VARNISH_ADMIN_LISTEN_PORT=6082
	# Shared secret file for admin interface
	VARNISH_SECRET_FILE=/etc/varnish/secret
	# Backend storage specification, see Storage Types in the varnishd(5)
	# man page for details.
	VARNISH_STORAGE="malloc,256M"               # 缓存位置和大小
	# User and group for the varnishd worker processes
	VARNISH_USER=varnish
	VARNISH_GROUP=varnish
	# Other options, see the man page varnishd(1)
	#DAEMON_OPTS="-p thread_pool_min=5 -p thread_pool_max=500 -p thread_pool_timeout=300"
```





## varnishd


```shell
varnishd [-a address[:port]] [-b host[:port]] [-C] [-d] [-f config]
      [-F]  [-g  group]  [-h type[,options]] [-i identity] [-l shl[,free[,fill]]] [-M address:port]
      [-n name] [-P file] [-p param=value] [-r  param[,param...]   [-s  [name=]kind[,options]]  [-S
      secret-file] [-T address[:port]] [-t ttl] [-u user] [-V]
```

　　选项解释

```
-a address[:port][,address[:port][...] 	默认6081 和 6082管理端口	
-d     	Enables debugging mode
-f config		vcl 配置文件
-s [name=]type[,options] 	指明使用哪一种存储机制： malloc[,size] 、 file[,path[,size[,granularity]]] 、 persistent,path,size
```

　　varnishadm所有子命令

```
help
200        
help [<command>]
ping [<timestamp>]
auth <response>
quit
banner
status
start
stop
vcl.load <configname> <filename>
vcl.inline <configname> <quoted_VCLstring>
vcl.use <configname>
vcl.discard <configname>
vcl.list
param.show [-l] [<param>]
param.set <param> <value>
panic.show
panic.clear
storage.list
vcl.show [-v] <configname>
backend.list [<backend_expression>]
backend.set_health <backend_expression> <state>
ban <field> <operator> <arg> [&& <field> <oper> <arg>]... 				# 清理缓存
ban.list
```


## purges清理缓存

　　varnish4清楚缓存方法主要有，通过varnishadm 管理，或vcl配置。其中vcl配置可以让客户端手动请求清楚缓存，以保证局部数据及时更新，而不用重启varnish服务器。

### PURGE请求清理缓存


``` shell
#允许清除缓存IP集
acl purge_ip{
    "127.0.0.1";
    "localhost";
}
sub vcl_recv {
   if(req.method ~ "PURGE"){
      if(client.ip ~ purge_ip){
          return(purge);//清除缓存
      }
      return (synth(404,"Not Found"));
   }
}
sub vcl_purge{
    return (synth(200,"success"));
}
```

### ban命令清理

　　缓存清理部分主要使用的是ban命令，在一些老的varnish版本里是purge命令。varnishadm ban相关的处理命令非常强大，支持正则和不同的域名进行区分，还支持按文件大小进行处理。

　　使用ban命令，需要调用varnishadm管理命令。查看varnishadm，可以进入管理命令行，再使用ban命令。

``` shell
[root@centos72 varnish]# varnishadm 
200        
-----------------------------
Varnish Cache CLI 1.0
-----------------------------
Linux,3.10.0-514.el7.x86_64,x86_64,-smalloc,-smalloc,-hcritbit
varnish-4.0.4 revision 386f712

Type 'help' for command list.
Type 'quit' to close CLI session.

help
200        
help [<command>]
ping [<timestamp>]
auth <response>
quit
banner
status
start
stop
vcl.load <configname> <filename>
vcl.inline <configname> <quoted_VCLstring>
vcl.use <configname>
vcl.discard <configname>
vcl.list
param.show [-l] [<param>]
param.set <param> <value>
panic.show
panic.clear
storage.list
vcl.show [-v] <configname>
backend.list [<backend_expression>]
backend.set_health <backend_expression> <state>
ban <field> <operator> <arg> [&& <field> <oper> <arg>]...
ban.list
```

1、清理所有域名下download下的缓存
``` shell
varnishadm -T 127.0.0.1:2000 ban.url /download/
```

2、清理example.com域名下所有png文件的缓存

``` shell
varnishadm -T 127.0.0.1:2000  ban req.http.host == "example.com" && req.url ~ ".png$"
```
3、以上是清理所有大于10MB的ogg文件

``` shell
varnishadm -T 127.0.0.1:2000 req.url !~ ".ogg$" && obj.size > 10MB
```
4、清理www.example.com还是example.com下的cookile值USERID=1663的所有缓存

``` shell
req.http.host ~ "^(?i)(www.)example.com$" && obj.http.set-cookie ~ "USERID=1663"
```

## varnish管理

1、varnishstat - varnish 

    -1(数字) 				列出所有field name
    -1 -f FILD_NAME		列出单个字段
    -l

2、varnishtop 	- varnish log entry ranking

    -i 						指定单个标签
    -x 						除开某个标签


## 完整示例


``` python
#
# This is an example VCL file for Varnish.
#
# It does not do anything by default, delegating control to the
# builtin VCL. The builtin VCL is called when there is no explicit
# return statement.
#
# See the VCL chapters in the Users Guide at https://www.varnish-cache.org/docs/
# and http://varnish-cache.org/trac/wiki/VCLExamples for more examples.
# Marker to tell the VCL compiler that this VCL has been adapted to the
# new 4.0 format.
vcl 4.0;
import directors;
probe backend_healthcheck { # 创建健康监测
    .url = /health.html;
    .window = 5;
    .threshold = 2;
    .interval = 3s;
}
backend web1 {    # 创建后端主机
    .host = "static1.lnmmp.com";
    .port = "80";
    .probe = backend_healthcheck;
}
backend web2 {
    .host = "static2.lnmmp.com";
    .port = "80";
    .probe = backend_healthcheck;
}
backend img1 {
    .host = "img1.lnmmp.com";
    .port = "80";
    .probe = backend_healthcheck;
}
backend img2 {
    .host = "img2.lnmmp.com";
    .port = "80";
    .probe = backend_healthcheck;
}
vcl_init {    # 创建后端主机组，即directors
    new web_cluster = directors.random();
    web_cluster.add_backend(web1);
    web_cluster.add_backend(web2);
    new img_cluster = directors.random();
    img_cluster.add_backend(img1);
    img_cluster.add_backend(img2);
}
acl purgers {    # 定义可访问来源IP
        "127.0.0.1";
        "192.168.0.0"/24;
}
sub vcl_recv {
    if (req.request == "GET" && req.http.cookie) {    # 带cookie首部的GET请求也缓存
        return(hash);
}
    if (req.url ~ "test.html") {    # test.html文件禁止缓存
        return(pass);
    }
    if (req.request == "PURGE") {    # PURGE请求的处理
        if (!client.ip ~ purgers) {
            return(synth(405,"Method not allowed"));
        }
        return(hash);
    }
    if (req.http.X-Forward-For) {    # 为发往后端主机的请求添加X-Forward-For首部
        set req.http.X-Forward-For = req.http.X-Forward-For + "," + client.ip;
    } else {
        set req.http.X-Forward-For = client.ip;
    }
    if (req.http.host ~ "(?i)^(www.)?lnmmp.com$") {    # 根据不同的访问域名，分发至不同的后端主机组
        set req.http.host = "www.lnmmp.com";
        set req.backend_hint = web_cluster.backend();
      } elsif (req.http.host ~ "(?i)^images.lnmmp.com$") {
            set req.backend_hint = img_cluster.backend();
      }
        return(hash);
    }
sub vcl_hit { # PURGE请求的处理
    if (req.request == "PURGE") {  
        purge;
        return(synth(200,"Purged"));
    }
}
sub vcl_miss {    # PURGE请求的处理
    if (req.request == "PURGE") {
        purge;
        return(synth(404,"Not in cache"));
    }
}
sub vcl_pass {    # PURGE请求的处理
    if (req.request == "PURGE") {
        return(synth(502,"PURGE on a passed object"));
    }
}
sub vcl_backend_response { # 自定义缓存文件的缓存时长，即TTL值
    if (req.url ~ "\.(jpg|jpeg|gif|png)$") {
        set beresp.ttl = 7200s;
    }
    if (req.url ~ "\.(html|css|js)$") {
        set beresp.ttl = 1200s;
    }
    if (beresp.http.Set-Cookie) { # 定义带Set-Cookie首部的后端响应不缓存，直接返回给客户端
        return(deliver);
    }
}
sub vcl_deliver {
    if (obj.hits > 0) {    # 为响应添加X-Cache首部，显示缓存是否命中
        set resp.http.X-Cache = "HIT from " + server.ip;
    } else {
        set resp.http.X-Cache = "MISS";
    }
}
``` 