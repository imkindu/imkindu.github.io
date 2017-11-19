---
title : SSH端口转发
date : 2017-09-13 09:00:00
author : imkindu
categories : linux
tags :
- ssh
---

　　ssh是个多用途的工具，不仅可以远程登录，还可以搭建socks代理、进行内网穿透，这是利用它的端口转发功能来实现的。
<!--more-->

---

## 概述

　　SSH除了可以实现密钥登录以外，还提供了一个非常实用的功能：端口转发。它能够将其他TCP端口的网络数据通过SSH链接来转发，并自动提供了加密和解密服务。这一过程我们称为`隧道（tunneling）`。如果平时工作环境中防火墙限制了某些端口的使用，但是允许SSH链接，就能通过将TCP端口转发来使用SSH通讯。


> SSH端口转发提供了两大功能：

>> ```shell
突破防火墙的限制，建立隧道，实现一些原本无法建立的TCP连接

加密SSH Client 和 SSH Server 之间的数据加密
```

## 分类

>本地转发       `ssh -L <local port>:<remote host>:<remote port> <SSH hostname>`
远程转发        `ssh -R <local port>:<remote host>:<remote port> <SSH hostname>`   
动态转发        `ssh -D <local port> <SSH Server>`

**选项**

>-f 后台启用
-N 不打开远程shell,处于等待状态,留在本机，不登录上代理服务器（默认会ssh上SSH Server）
-g 启用网关功能

---

### 本地转发

<div align=center>
![](/images/openssh/sshben.jpg)
</div>

> `ssh -L <local port>:<remote host>:<remote port> <SSH hostname>`
本地端口:目标主机:目标主机端口  SSH server
`ssh -L 9527:172.18.56.150:23 -Nf 172.18.251.20`

　　用户访问不了内部服务器，但是其中有一台服务器可以让我们访问，我们可以将这台服务器看做`代理服务器`，这台代理服务器是可以和内部服务器连接。注意：非管理员账号是无权绑定`1-1023`端口的，所以一般选用`1024-65535`之间尚未使用的端口号。

　　这里`9527`是本地监听端口，在SSH Server上会随机监听一个端口，建议一个隧道。所有发送到本机9527端口的数据，都会通过这一条隧道走向SSH Server 并发送到远程remote主机port端口。

> 172.18.56.69 本地主机

<div align=center>
![](/images/openssh/01.png)
</div>

本机监听9527开启了一个端口，和SSH Server建立隧道，使用了ssh 22端口

> 172.18.251.20 SSH Server

<div align=center>
![](/images/openssh/02.png)
</div>

SSH Server 开启了22端口和 本地主机 56.69建立隧道，在隧道建立后，用户telnet 上远程主机时，会开启一个随机端口和远程主机建立连接

> 172.18.56.150 远程主机

<div align=center>
![](/images/openssh/03.png)
</div>

本地主机telnet上时，远程主机开启23端口和 SSH Server响应

> **总结：**
>>本地端口转发，是在本地主机操作，和SSH Server建立隧道，两边必须都有SSH，SSH Server是一个中间代理作业，会临时开启两个随机端口，响应两边的请求。同时，本地主机转发是一个`点对点，端口对端口`的转发。

---

### 远程转发
<div align=center>
![](/images/openssh/SSHyuan.jpg)
</div>

>`ssh -R <local port>:<remote host>:<remote port> <SSH hostname>`
远程主机端口:目标主机:目标主机端口  远程主机
`ssh -L 9527:172.18.56.150:23 -Nf 172.18.56.69`

　　用户访问不了所有服务器，但是SSH Server可以访问外网，连接到本地主机，同时可以和目录主机连接。此时，我们所在的位置就是SSH Server。

---

### 动态转发
<div align=center>
![](/images/openssh/sshdong.jpg)
</div>

>`ssh -D <local port> <SSH Server>`
本地端口:目标主机:目标主机端口
`ssh -f -N -D 1080 root@172.18.251.20 `

　　不管是本地转发，还是远程转发，都是`点对点`的转发，如翻墙工具，我们希望的是本机指定端口，可以访问所有外网，这是`点对面`的需求，动态转发就实现了这种效果。不管我们访问哪个目标主机任意端口，通通由远程代理主机处理。

---

## 后台进程处理

>通过ssh建立的隧道，如果放在后台执行，要关闭这个隧道连接，只能Kill进程。

---

## 参考

[阮一峰 SSH原理与运用（二）：远程操作与端口转发](http://www.ruanyifeng.com/blog/2011/12/ssh_port_forwarding.html)
[IBM 实战SSH端口转发](https://www.ibm.com/developerworks/cn/linux/l-cn-sshforward)


备注: 时间紧张，此篇博客补全，稍后补上。