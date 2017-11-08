---
title: tomcat redis-session-manager
author : imkindu
date : 2017-11-07 09:00:00
categories: java
tags:
  - tomcat
  - java
  - redis-session-manager
---

　　分布式web server集群部署后需要实现session共享，针对 tomcat 服务器的实现方案多种多样，比如 tomcat cluster session 广播、nginx IP hash策略、nginx sticky module等方案，本文主要介绍了使用 redis 服务器进行 session 统一存储管理的共享方案。


<!--more-->

　　github提供了多个版本的redis-sessiom-manager版本，大致思路都一致，需要特别注意版本区别和配置文件编写。

https://github.com/search?utf8=%E2%9C%93&q=tomcat-redis-session-manager&type=

## 架构图

![](/images/java/101.png)


## jar依赖包

添加redis session集群依赖的jar包到 TOMCAT_BASE/lib 目录下

- tomcat-redis-session-manager-${VERSION}.jar
- jedis-2.5.2.jar
- commons-pool2-2.2.jar

jar包下载 : https://pan.baidu.com/s/1bokMOVH

将下载jar类库包放入`tomcat/lib/`


## 配置context.xml


``` xml
　 <!-- host="192.168.159.129"       Redis地址 -->
    <!-- port="6379"                  Redis端口 -->
    <!-- password="123456"            Redis密码 -->
    <!-- database="0"                 存储Session的Redis库编号 -->
    <!-- maxInactiveInterval="60"     Session失效的间隔（秒） -->
    
    <Valve className="com.orangefunction.tomcat.redissessions.RedisSessionHandlerValve" />        
    <Manager className="com.orangefunction.tomcat.redissessions.RedisSessionManager" 
        host="192.168.159.129"       
        port="6379"                 
        password="123456"            
        database="0"                 
        maxInactiveInterval="60" />
```

## 属性解释

`host` : redis服务器地址

`port` : redis服务器的端口号

`database` : 要使用的redis数据库索引

`maxInactiveInterval` : session最大空闲超时时间，如果不填则使用tomcat的超时时长，一般tomcat默认为1800 即半个小时

`sessionPersistPolicies` : session保存策略，除了默认的策略还可以选择的策略有：

- `[SAVE_ON_CHANGE]` : 每次 session.setAttribute() 、 session.removeAttribute() 触发都会保存. 
	注意：此功能无法检测已经存在redis的特定属性的变化，
	权衡：这种策略会略微降低会话的性能，任何改变都会保存到redis中.

- `[ALWAYS_SAVE_AFTER_REQUEST]`: 每一个request请求后都强制保存，无论是否检测到变化.
	注意：对于更改一个已经存储在redis中的会话属性，该选项特别有用. 
	权衡：如果不是所有的request请求都要求改变会话属性的话不推荐使用，因为会增加并发竞争的情况。

`sentinelMaster` :	redis集群主节点名称（Redis集群是以分片(Sharding)加主从的方式搭建，满足可扩展性的要求）

`sentinels` :	redis集群列表配置(类似zookeeper，通过多个Sentinel来提高系统的可用性)

`connectionPoolMaxTotal`

`connectionPoolMaxIdle`	jedis最大能够保持idel状态的连接数

`connectionPoolMinIdle`	与connectionPoolMaxIdle相反

`maxWaitMillis`	jedis池没有对象返回时，最大等待时间

`minEvictableIdleTimeMillis`

`softMinEvictableIdleTimeMillis`

`numTestsPerEvictionRun`

`testOnCreate`

`testOnBorrow`	jedis调用borrowObject方法时，是否进行有效检查

`testOnReturn`	jedis调用returnObject方法时，是否进行有效检查

`testWhileIdle`

`timeBetweenEvictionRunsMillis`

`evictionPolicyClassName`

`blockWhenExhausted`

`jmxEnabled`

`jmxNameBase`

`jmxNamePrefix`

