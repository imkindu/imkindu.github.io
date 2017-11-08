---
title: tomcat集群
author : imkindu
date : 2017-11-05 09:00:00
layout: post 
categories: java
tags:
  - tomcat
  - java
---

　　tomcat默认使用普通用户启动，而非管理员，启动端口是`8080`，在企业中，一般不直接对外提供服务，前端使用apache或者nginx做代理。

<!--more-->

## 环境准备

　　为做tomcat集群环境，特先准备好两台tomcat服务器和一台apache、nginx服务器

![](/images/java/100.png)

## 安装solo

　　Solo 是款专业、简约、稳定、极速的 Java 开源博客系统，http://b3log.org/


### tomcat-solo虚拟机

``` xml
<Host name="solo.imkindu.cn" appBase="webapps/solo" unpackWARs="true" autoDeploy="true" >
    <Context path="/" docBase="ROOT" />
</Host>
```

### latke.properties

　　配置solo时，需要修改latke.properties对用浏览器使用地址，否则容易出现样式等问题。

``` shell
#### Server ####
# Browser visit protocol
serverScheme=http
# Browser visit domain name
serverHost=solo.imkindu.cn
# Browser visit port, 80 as usual, THIS IS NOT SERVER LISTEN PORT!
serverPort=80
```

### local.properties

　　solo默认使用H2数据库，但是个人不是很熟悉，特配置改成存入mariadb

``` shell
#### MySQL runtime ####
runtimeDatabase=MYSQL
jdbc.username=root
jdbc.password=root
jdbc.driver=com.mysql.jdbc.Driver
jdbc.URL=jdbc:mysql://192.168.56.55:3306/solo?useUnicode=yes&characterEncoding=utf8
jdbc.pool=druid
```

<div aling="center">
![](/images/java/solo.png)
</div>





## session集群

　　利用solo做集群会话保持之前，先来聊聊session集群的几种模式。

session会话保持主要有三种模式：

``` python
session sticky  会话绑定
	source_ip
		nginx : ip_hash
		haproxy : source
		lvs : sh
	cookie 
		nginx : hash 
		haproxy : cookie 

session cluster 会话集群
	tomcat deltamanager
		httpd + tomcat cluster
			httpd : mod_proxy, mod_proxy_http, mod_proxy_balancer
		httpd + tomcat cluster
			httpd : mod_proxy, mod_proxy_ajp, mod_proxy_balancer
		httpd + tomcat cluster
			httpd : mod_jk
		nginx + tomcat cluster
		
session server 会话服务器
	memcached, redis
```

## apache代理


　　使用apache代理，主要使用到了httpd的`proxy_module`、`proxy_httpd_module`、`proxy_ajp_module`、`proxy_balacer_module`几个模块 


``` shell
[root@centos71 conf.d]# httpd -M | egrep proxy
 proxy_module (shared)
 proxy_ajp_module (shared)
 proxy_balancer_module (shared)
 proxy_connect_module (shared)
 proxy_express_module (shared)
 proxy_fcgi_module (shared)
 proxy_fdpass_module (shared)
 proxy_ftp_module (shared)
 proxy_http_module (shared)
 proxy_scgi_module (shared)
 proxy_wstunnel_module (shared)
```



### apache指令说明

关于如上apache指令的说明：


- `ProxyPreserveHost {On|Off}` ：如果启用此功能，代理会将用户请求报文中的Host:行发送给后端的服务器，而不再使用ProxyPass指定的服务器地址。如果想在反向代理中支持虚拟主机，则需要开启此项，否则就无需打开此功能。


- `ProxyVia  {On|Off|Full|Block}` ：用于控制在http首部是否使用Via:，主要用于在多级代理中控制代理请求的流向。默认为Off，即不启用此功能；`On`表示每个请求和响应报文均添加Via:；`Full`表示每个Via:行都会添加当前apache服务器的版本号信息；`Block`表示每个代理请求报文中的Via：都会被移除。


- `ProxyRequests {On|Off}` ：是否开启apache正向代理的功能；启用此项时为了代理http协议必须启用mod_proxy_http模块。同时，如果为apache设置了ProxyPass，则必须将ProxyRequests设置为Off。


- `ProxyPass  [path]  !|url  [key=value key=value ...]]` ：将后端服务器某URL与当前服务器的某虚拟路径关联起来作为提供服务的路径，path为当前服务器上的某虚拟路径，url为后端服务器上某URL路径。使用此指令时必须将ProxyRequests的值设置为Off。需要注意的是，如果path以“/”结尾，则对应的url也必须以“/”结尾，反之亦然。


- `ProxyPassReverse` ：用于让apache调整HTTP重定向响应报文中的Location、Content-Location及URI标签所对应的URL，在反向代理环境中必须使用此指令避免重定向报文绕过proxy服务器。



另外，mod_proxy模块在httpd2.1的版本之后支持与后端服务器的连接池功能，连接在按需创建在可以保存至连接池中以备进一步使用。连接池大小或其它设定可以通过在ProxyPass中使用key=value的方式定义。常用的key如下所示：

>◇ min：连接池的最小容量，此值与实际连接个数无关，仅表示连接池最小要初始化的空间大小。
◇ max：连接池的最大容量，每个MPM都有自己独立的容量；都值与MPM本身有关，如Prefork的总是为1，而其它的则取决于ThreadsPerChild指令的值。
◇ loadfactor：用于负载均衡集群配置中，定义对应后端服务器的权重，取值范围为1-100。
◇ retry：当apache将请求发送至后端服务器得到错误响应时等待多长时间以后再重试。单位是秒钟。

**`如果Proxy指定是以balancer://开头，即用于负载均衡集群时`**

其还可以接受一些特殊的参数，如下所示：

>◇lbmethod：apache实现负载均衡的调度方法，默认是byrequests，即基于权重将统计请求个数进行调度，bytraffic则执行基于权重的流量计数调度，bybusyness通过考量每个后端服务器的当前负载进行调度。
◇ maxattempts：放弃请求之前实现故障转移的次数，默认为1，其最大值不应该大于总的节点数。
◇ nofailover：取值为On或Off，设置为On时表示后端服务器故障时，用户的session将损坏；因此，在后端服务器不支持session复制时可将其设置为On。
◇ stickysession：调度器的sticky session的名字，根据web程序语言的不同，其值为JSESSIONID或PHPSESSIONID。


上述指令除了能在banlancer://或ProxyPass中设定之外，也可使用ProxySet指令直接进行设置，如：

``` shell
<Proxy balancer://hotcluster>
BalancerMember  http://www1.magedu.com:8080 loadfactor=1
BalancerMember  http://www2.magedu.com:8080 loadfactor=2
ProxySet  lbmethod=bytraffic
</Proxy>
```

### **使用http协议直接代理tomcat**

``` python
<VirtualHost *:80>
	ServerName solo.imkindu.cn
	ProxyVia On
	ProxyRequests Off			      # 关闭正向代理
	ProxyPreserveHost On			      # host主机名
	<Proxy *>
		Require all granted
	</Proxy>
	ProxyPass / http://192.168.56.52:8080/
	ProxyPassReverse / http://192.168.56.52:8080/	
	<Location />
		Require all granted
	</Location>
</VirtualHost>
```

<div align="center">
![](/images/java/006.png)
</div>



### **使用ajp协议代理tomcat**

　　使用apache2.4和ajp协议代理tomcat需要使用到httpd的 `proxy_ajp_module`

使用`httpd -M`查看是否编译了proxy_ajp_module

```
[root@centos71 conf.d]# httpd -M | grep --color proxy_ajp_module 
 proxy_ajp_module (shared)
 ```

```
<VirtualHost *:80>
	ServerName solo.imkindu.cn
	ProxyVia On
	ProxyRequests Off		    # 关闭正向代理
	ProxyPreserveHost On		    # host主机名
	<Proxy *>
		Require all granted
	</Proxy>
	ProxyPass / ajp://172.18.56.52:8009/ 	# tomcat ajp默认使用8009端口
	ProxyPassReverse / ajp://172.18.56.52:8009/	
	<Location />
		Require all granted
	</Location>
</VirtualHost>
```
---


### apache负载均衡tomcat


　　使用apache 负载调度tomcat 需要使用到 `mod_proxy_balacer`模块

　　详细手册查看 ： http://httpd.apache.org/docs/2.4/mod/mod_proxy_balancer.html


**参数详解**




- `<proxy>` 应用于代理资源的指令的容器

如果Proxy指定是以balancer://开头，即用于负载均衡集群

> Description:	Container for directives applied to proxied resources 
Syntax:	Proxy wildcard-url ...  Proxy
Context:	server config, virtual host
Status:	Extension
Module:	mod_proxy


- `BalancerMember` Add a member to a load balancing group 添加成员

> Syntax:	BalancerMember [balancerurl] url [key=value [key=value ...]] 可以添加额外参数
参数详解 ： http://httpd.apache.org/docs/2.4/mod/mod_proxy.html#proxypass


|   参数   |  解释    |
| ---- | ---- |
|  loadfactor    |   权重 1-100    |
|   route   |    	Route of the worker when used inside load balancer. The route is a value appended to session id.后台tomcat的engine 可以添加一个 jvmRoute="Tomcat52" 与之对应 `Engine name="Catalina" defaultHost="localhost" jvmRoute="Tomcat52"`|




其还可以接受一些特殊的参数，如下所示：

>◇ `lbmethod` ：apache实现负载均衡的调度方法，默认是byrequests，即基于权重将统计请求个数进行调度，bytraffic则执行基于权重的流量计数调度，bybusyness通过考量每个后端服务器的当前负载进行调度。
◇ `maxattempts` ：放弃请求之前实现故障转移的次数，默认为1，其最大值不应该大于总的节点数。
◇ `nofailover`：取值为On或Off，设置为On时表示后端服务器故障时，用户的session将损坏；因此，在后端服务器不支持session复制时可将其设置为On。
◇ `stickysession`：调度器的sticky session的名字，根据web程序语言的不同，其值为JSESSIONID或PHPSESSIONID。


``` python
<proxy balancer://tomcat>
        BalancerMember http://192.168.56.51:8080 loadfactor=1 route=Tomcat51
        BalancerMember http://192.168.56.52:8080 loadfactor=1 route=Tomcat52
        ProxySet  lbmethod=byrequests 								 		# 添加调度算法 
</proxy>

<VirtualHost *:80>
        #ServerName solo.imkindu.cn
        ServerName 172.18.56.71
        ProxyVia On
        ProxyRequests Off               # 关闭反代
        ProxyPreserveHost On            # host主机名
        <Proxy *>
                Require all granted
        </Proxy>
        ProxyPass / balancer://tomcat/
        ProxyPassReverse / balance://tomcat/
        <Location />
                Require all granted
        </Location>
</VirtualHost>
```


### 负载tomcat并保持会话


　　`stickysession`：调度器的sticky session的名字，根据web程序语言的不同，其值为JSESSIONID或PHPSESSIONID。利用`stickysession`和route保持会话。


``` python
# 注释
Header add Set-Cookie "ROUTEID=.%{BALANCER_WORKER_ROUTE}e; path=/" env=BALANCER_ROUTE_CHANGED
<proxy balancer://tomcat>
	BalancerMember http://192.168.56.51:8080 loadfactor=1 route=Tomcat51
	BalancerMember http://192.168.56.52:8080 loadfactor=1 route=Tomcat52	
	ProxySet stickysession=ROUTEID  							# stickysession 利用route标识保持session会话
</proxy>

<VirtualHost *:80>
	#ServerName solo.imkindu.cn
	ServerName 172.18.56.71
	ProxyVia On
	ProxyRequests Off		# 关闭反代
	ProxyPreserveHost On		# host主机名
	<Proxy *>
		Require all granted
	</Proxy>
	ProxyPass / balancer://tomcat/
	ProxyPassReverse / balance://tomcat/	
	<Location />
		Require all granted
	</Location>
</VirtualHost>
```






### balancer管理页面

　　httpd2.4负载均衡模块balancer自带了一个`balancer-manager`管理界面，可以通过此界面查看负载情况，并设置单个后台backend等。

详细参考查看httpd官方proxy_balancer模块 ： http://httpd.apache.org/docs/2.4/mod/mod_proxy_balancer.html

``` shell
Header add Set-Cookie "ROUTEID=.%{BALANCER_WORKER_ROUTE}e; path=/" env=BALANCER_ROUTE_CHANGED
<proxy balancer://tomcat>
	BalancerMember http://192.168.56.51:8080 loadfactor=1 route=Tomcat51
	BalancerMember http://192.168.56.52:8080 loadfactor=1 route=Tomcat52	
	ProxySet stickysession=ROUTEID
</proxy>
<VirtualHost *:80>
	ServerName solo.imkindu.cn
	ProxyVia On
	ProxyRequests Off		
	ProxyPreserveHost On		# host主机名
	<Proxy *>
		Require all granted
	</Proxy>
	ProxyPass / balancer://tomcat/
	ProxyPassReverse / balance://tomcat/	
	<Location />
		Require all granted
	</Location>
	<Location "/balancer-manager">			# balancer-manager
    		SetHandler balancer-manager		# 
		ProxyPass ! 						# 不使用代理
	   	Require all granted
	</Location>
</VirtualHost>
```

![](/images/java/manager.png)


---


## nginx代理tomcat

　　一般真正使用的还是nginx前端负载均衡，后端在tomcat服务器上再次使用Nginx代理到本机的tomcat，这样充分发挥调度器、nginx的动静分离、Tomcat只处理jsp，保证了各自模块的高效性能。


### nginx直接代理到tomcat8080

nginx代理服务器配置

``` shell
server {
	listen 80;
	server_name solo.imkindu.cn;
	location / {
		proxy_pass http://192.168.56.52:8080/;
		proxy_set_header Host $host:$server_port;
		proxy_set_header X-Real_IP $remote_addr;
	}
}
```

### nginx代理本机tomcat

　　通过nginx代理服务器代理到 tomcat服务器上的nginx，再次转发到tomcat

tomcat上nginx配置：

``` shell
upstream solo {
	server localhost:8080 ;
}

server {
	listen 80;
	server_name solo.imkindu.cn;
	location / {
		proxy_pass http://solo$request_uri;
		proxy_set_header Host $host:$server_port;
		proxy_set_header X-Real_IP $remote_addr;
	}
}
```




## session cluster

　　session 集群，其主要工作原理是：将多台后端数据服务器作为一个session集群，常用的有`deltamanager`这种将任意一节点的会话信息同步到其他所有节点，保证不论前台调度器将请求调度到哪台服务器都可以获取会话信息，`BackupManager`模式：将多台服务器分为多个小组，其中小组之间的多个服务器同步会话信息，不用同步到所有节点，当小组任意节点挂了之后，去其他小组节点获取会话信息。

### deltaManager

　　tomcat默认的集群会话管理器——DeltaManager。它主要用于集群中各个节点之间会话状态的同步维护。DeltaManager的职责是`将某节点的会话该变同步到集群内其他成员节点上`，它属于全节点复制模式，所谓全节点复制是指集群中某个节点的状态变化后需要同步到集群中剩余的节点，非全节点方式可能只是同步到其中某个或若干节点。

<div align="center">
![](/images/java/010.png)
</div>



查看帮助：`Clustering` http://tomcat.apache.org/tomcat-7.0-doc/cluster-howto.html


　　1、使用deltamanager这种session集群模式，需要在`server.xml`中配置对应多播地址、监听端口等，deltamanager可以配置在 `<Engine>` or  `<Host>`中。

<div align="center">
![](/images/java/200.png)
</div>


**注意** ：官方文档配置文件中 `ClusterListener` 两行没有闭合标签"`/`"

``` xml
<Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"
				channelSendOptions="8">
		<Manager className="org.apache.catalina.ha.session.DeltaManager"
				expireSessionsOnShutdown="false"
				notifyListenersOnReplication="true"/>
		<Channel className="org.apache.catalina.tribes.group.GroupChannel">
				<Membership className="org.apache.catalina.tribes.membership.McastService"
						address="228.0.56.71"
						port="45564"
						frequency="500"
						dropTime="3000"/>
				<Receiver className="org.apache.catalina.tribes.transport.nio.NioReceiver"
						address="192.168.56.51"
						port="4000"
						autoBind="100"
						selectorTimeout="5000"
						maxThreads="6"/>
				<Sender className="org.apache.catalina.tribes.transport.ReplicationTransmitter">
						<Transport className="org.apache.catalina.tribes.transport.nio.PooledParallelSender"/>
				</Sender>
				<Interceptor className="org.apache.catalina.tribes.group.interceptors.TcpFailureDetector"/>
				<Interceptor className="org.apache.catalina.tribes.group.interceptors.MessageDispatch15Interceptor"/>
		</Channel>
		<Valve className="org.apache.catalina.ha.tcp.ReplicationValve"
				filter=""/>
		<Valve className="org.apache.catalina.ha.session.JvmRouteBinderValve"/>
		<Deployer className="org.apache.catalina.ha.deploy.FarmWarDeployer"
				tempDir="/tmp/war-temp/"
				deployDir="/tmp/war-deploy/"
				watchDir="/tmp/war-listen/"
				watchEnabled="false"/>
		<ClusterListener className="org.apache.catalina.ha.session.JvmRouteSessionIDBinderListener" />
		<ClusterListener className="org.apache.catalina.ha.session.ClusterSessionListener" />
</Cluster>
```


选项解释：

`Membership` ：MemberShip组件自动检索发现集群里的新节点或已经停止工作的节点，并发出相应的通知。默认使用组播（Multicast）实现。 

`Receiver` ：负责监听接收其他节点传送过来的数据。默认使用non-blocking TCP Server sockets。

`Sender` ： 发送数据给其他节点。管理从一个节点发送到另外一个节点的出站连接和数据信息，允许信息并行发送。默认使用TCP Client Sockets。 


　　2、确保web.xml有`<distributable/>`标签 Make sure your web.xml has the <distributable/> element 

<div align="center">
![](/images/java/melta.png)
</div>

``` xml
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://java.sun.com/xml/ns/javaee
                      http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
  version="3.0">

<distributable/> 	# 添加这个标签即可
```

查看效果：

　　无论请求调度到哪一台，每台服务器都保存了所有session会话信息。

<div align="center">
![](/images/java/51.png)
</div>
<div align="center">
![](/images/java/52.png)
</div>



### BackupManager

　　backupManager同deltaManager，只不过是将`Manager`换成了`BackupManager`，然后定义多个小组，每个小组节点使用同一个多播地址，虽然backupmanager比deltamanager效率上提高了，但是想想有一个问题，当前端调度器将请求调度到了另外一台小组成员外的节点，那不就是获取不到会话信息了。当集群中的节点数量很多并且部署着不同应用时，可以使用BackupManager，BackManager仅向部署了当前应用的节点拷贝Session。但是到目前为止BackupManager并未经过大规模测试，可靠性不及DeltaManager。


``` xml
<Cluster className="org.apache.catalina.ha.tcp.SimpleTcpCluster"
			channelSendOptions="8">
	<Manager className="org.apache.catalina.ha.session.BackupManager" # 注意这里更换为BackupManager
			expireSessionsOnShutdown="false"
			notifyListenersOnReplication="true"/>
	<Channel className="org.apache.catalina.tribes.group.GroupChannel">
			<Membership className="org.apache.catalina.tribes.membership.McastService"
					address="228.0.56.71"
					port="45564"
					frequency="500"
					dropTime="3000"/>
			<Receiver className="org.apache.catalina.tribes.transport.nio.NioReceiver"
					address="192.168.56.51"
					port="4000"
					autoBind="100"
					selectorTimeout="5000"
					maxThreads="6"/>
			<Sender className="org.apache.catalina.tribes.transport.ReplicationTransmitter">
					<Transport className="org.apache.catalina.tribes.transport.nio.PooledParallelSender"/>
			</Sender>
			<Interceptor className="org.apache.catalina.tribes.group.interceptors.TcpFailureDetector"/>
			<Interceptor className="org.apache.catalina.tribes.group.interceptors.MessageDispatch15Interceptor"/>
	</Channel>
	<Valve className="org.apache.catalina.ha.tcp.ReplicationValve"
			filter=""/>
	<Valve className="org.apache.catalina.ha.session.JvmRouteBinderValve"/>
	<Deployer className="org.apache.catalina.ha.deploy.FarmWarDeployer"
			tempDir="/tmp/war-temp/"
			deployDir="/tmp/war-deploy/"
			watchDir="/tmp/war-listen/"
			watchEnabled="false"/>
	<ClusterListener className="org.apache.catalina.ha.session.JvmRouteSessionIDBinderListener" />
	<ClusterListener className="org.apache.catalina.ha.session.ClusterSessionListener" />
</Cluster>
```


## session server

　　在企业中用的最多的应该还是session server服务器这种模式吧，配置也将为简单，其原理就是：将多个节点的会话信息同一保存至一台session服务器，常见的有将多个节点session保存至同一台memcached中，这样前台调度器和backend server都不会做任何配置了。而且memcached是基于内存缓存的，非常高效。


<div align="center">
![](/images/java/mem.png)
</div>


### MSM

　　tomcat会话存入memcached需要依赖于第三方类库，在github上有第三方提供，使用量最高的是`magro/memcached-session-manager`。

　　https://github.com/magro/memcached-session-manager

　　MSM(memcached-session-manager)支持tomcat6和tomcat7 ，利用Value（Tomcat 阀对Request进行跟踪。Request请求到来时，从memcached加载session，Request请求结束时，将tomcat session更新至memcached，以达到session共享之目的，支持sticky和 non-sticky 模式。

　　`Sticky 模式`：tomcat session为主session， memcached为备session。Request请求到来时， 从memcached加载备session到tomcat (仅当tomcat jvmroute发生变化时，否则直接取tomcat session)；Request请求结束时，将tomcat session更新至memcached，以达到主备同步之目的。

　　`Non-Sticky模式`：tomcat session为中转session， memcached1为主session，memcached 2为备session。Request请求到来时，从memcached2加载备session到tomcat，当容器中还是没有session 则从memcached1加载主session到tomcat，这种情况是只有一个memcached节点，或者有memcached1出错时，Request请求结束时，将tomcat session更新至主memcached1和备memcached2，并且清除tomcat session 。以达到主备同步之目的。


### SetupAndConfiguration wiki page

　　MSM提供了wiki，示例解释了多样性将tomcat的session存入memcached或者redis。wiki page 地址 ：https://github.com/magro/memcached-session-manager/wiki/SetupAndConfiguration

<div align="center">
![](/images/java/wiki.png)
</div>


### Configure tomcat

　　The configuration of tomcat requires two things: `you need to drop some jars in your $CATALINA_HOME/lib/` and WEB-INF/lib/ directories and you have to configure the memcached session manager in the related <Context> element (e.g. in META-INF/context.xml inside the application files).

### Add memcached-session-manager jars to tomcat

<div align="center">
![](/images/java/msm.png)
</div>

　　tomcat的lib目录如果是Yum安装的在`/usr/share/java/tomcat/`

``` shell
[ root@centos51 conf ]# ll /usr/share/tomcat/
total 8
drwxr-xr-x 2 root   root   4096 Oct 21 06:32 bin
lrwxrwxrwx 1 root   tomcat   11 Oct 21 06:32 conf -> /etc/tomcat
lrwxrwxrwx 1 root   tomcat   22 Oct 21 06:32 lib -> /usr/share/java/tomcat # 类库目录
lrwxrwxrwx 1 root   tomcat   15 Oct 21 06:32 logs -> /var/log/tomcat
-rw-r--r-- 1 tomcat tomcat 3952 Oct 22 19:39 solo.log
lrwxrwxrwx 1 root   tomcat   22 Oct 21 06:32 temp -> /var/cache/tomcat/temp
lrwxrwxrwx 1 root   tomcat   23 Oct 21 06:32 webapps -> /var/lib/tomcat/webapps
lrwxrwxrwx 1 root   tomcat   22 Oct 21 06:32 work -> /var/cache/tomcat/work
```


### Add custom serializers to your webapp

　　因为memcached只支持字符串这种可流式化数据,对于session对象这种数据，需要一些序列化工具，类似于咱们php经常弄的json_encode一样，将对象转换成字符串。常用的序列化工具有：

`kryo-serializer`: [msm-kryo-serializer](http://repo1.maven.org/maven2/de/javakaffee/msm/msm-kryo-serializer/), [kryo-serializers-0.34+](http://repo1.maven.org/maven2/de/javakaffee/kryo-serializers/), [kryo-3.x](http://repo1.maven.org/maven2/com/esotericsoftware/kryo/), [minlog](http://repo1.maven.org/maven2/com/esotericsoftware/minlog/), [reflectasm](http://repo1.maven.org/maven2/com/esotericsoftware/reflectasm/), [asm-5.x](http://repo1.maven.org/maven2/org/ow2/asm/asm/), [objenesis-2.x](http://repo1.maven.org/maven2/org/objenesis/objenesis/)
`javolution-serializer`: msm-javolution-serializer, javolution-5.4.3.1
`xstream-serializer`: msm-xstream-serializer, xstream, xmlpull, xpp3_min
`flexjson-serializer`: msm-flexjson-serializer, flexjson

<div align="center">
![](/images/java/jar.png)
</div>


### Configure  <Context> 

　　`Update the <Context> element (in META-INF/context.xml` or what else you choose for context definition, please check the related tomcat documentation for this) so that it contains the Manager configuration for the memcached-session-manager, like in the examples below.

　　github上提供了多个示例。

　　http://mvnrepository.com/artifact/joda-time/joda-time/1.6.2

　　**javolution序列化工具**

``` xml
<Host name="localhost"  appBase="webapps" 
		unpackWARs="true" autoDeploy="true" >
	<Context path="/" docBase="ROOT" reloadable="true">
		<Manager className="de.javakaffee.web.msm.MemcachedBackupSessionManager"
			memcachedNodes="n1:172.18.56.51:11211,n2:172.18.56.52:11211"
			failoverNodes="n2"
			# sticky="false" 			# 这里需要特别注意，最好不好加
			sessionBackupAsync= "false"   
			sessionBackupTimeout= "100"  
			copyCollectionsForSerialization="true"
			requestUriIgnorePattern=".*\.(ico|png|gif|jpg|css|js)$"
			transcoderFactoryClass="de.javakaffee.web.msm.serializer.javolution.JavolutionTranscoderFactory"
		/>
	</Context>
	<Valve className="org.apache.catalina.valves.AccessLogValve" directory="logs"
		prefix="localhost_access_log." suffix=".txt"
		pattern="%h %l %u %t &quot;%r&quot; %s %b" />
</Host>

```

Example for non-sticky sessions + kryo

``` xml
<Context>
  ...
  <Manager className="de.javakaffee.web.msm.MemcachedBackupSessionManager"
    memcachedNodes="n1:host1.yourdomain.com:11211,n2:host2.yourdomain.com:11211"
    sticky="false"
    sessionBackupAsync="false"
    lockingMode="uriPattern:/path1|/path2"
    requestUriIgnorePattern=".*\.(ico|png|gif|jpg|css|js)$"
    transcoderFactoryClass="de.javakaffee.web.msm.serializer.kryo.KryoTranscoderFactory"
    />
</Context>
```