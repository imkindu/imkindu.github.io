---
title : ansible第二篇：简单使用
date : 2017-11-17 19:56:00
author : imkindu
categories :
- devops
tags :
- ansible
---

　　ansible的简单使用介绍，其核心思想就是：利用ansible的各核心模块实现不同的管理功能。

<!--more-->


## 环境说明

　　三台主机：

`管理端`   ： 192.168.56.80
`管理节点` :  192.168.56.81  ~  192.168.56.82 ~ 192.168.56.83

 

## 生成ssh密钥对


　　在开始使用ansible前，先将环境准备好，生成ssh密钥对，实现免密码登录。

``` shell
[root@centos80 ~]# ssh-keygen -t rsa -P ""
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa): 
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
51:8a:60:43:57:11:d2:48:67:a8:bb:e8:46:d4:14:b5 root@centos80
The key's randomart image is:
+--[ RSA 2048]----+
|   .*+==*o.      |
|   ..+o*.o       |
|   o .E o        |
|  . o    .       |
| .   .  S        |
|  . .            |
| . . .           |
|  o .            |
| o.              |
+-----------------+
[root@centos80 ~]# ll -a .ssh/
total 16
drwx------  2 root root   57 Nov 16 16:36 .
dr-xr-x---. 7 root root 4096 Nov 16 16:08 ..
-rw-------  1 root root 1675 Nov 16 16:36 id_rsa
-rw-r--r--  1 root root  395 Nov 16 16:36 id_rsa.pub
-rw-r--r--  1 root root  174 Nov 14 10:53 known_hosts
```


``` shell
[root@centos80 ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub root@172.18.56.81
[root@centos80 ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub root@172.18.56.82
[root@centos80 ~]# ssh-copy-id -i ~/.ssh/id_rsa.pub root@172.18.56.83
```

## ansible.cfg配置文件

　　ansible配置文件默认是`/etc/ansible/ansible.cfg`，此文件基本全是模式配置，可以按需修改，默认都被注释。

``` shell
[root@centos80 ansible]# less ansible.cfg 
# config file for ansible -- http://ansible.com/
# ==============================================

# nearly all parameters can be overridden in ansible-playbook
# or with command line flags. ansible will read ANSIBLE_CONFIG,
# ansible.cfg in the current working directory, .ansible.cfg in
# the home directory or /etc/ansible/ansible.cfg, whichever it
# finds first

[defaults]

# some basic default values...

#inventory      = /etc/ansible/hosts            # 主机清单
#library        = /usr/share/my_modules/
#remote_tmp     = ~/.ansible/tmp
#local_tmp      = ~/.ansible/tmp
#forks          = 5                             # 每次并发多少个连接
#poll_interval  = 15
#sudo_user      = root                          # 默认管理账号
#ask_sudo_pass = True
#ask_pass      = True
#transport      = smart
#remote_port    = 22                            # 默认管理端口
#module_lang    = C
#module_set_locale = False
```


## ansible语法

> ansible -i /etc/ansible/host <host-pattern> [options]

　　其中-i用来指定inventory文件，默认就是使用/etc/ansible/hosts，其中all是针对hosts定义的所有主机执行，这里也可以指定hosts中定义的组名或模式。

``` shell
[root@centos80 ~]# ansible --help
Usage: ansible <host-pattern> [options]

-m：指定模块(默认是command模块)
-a：指定模块的参数
-u：指定执行远程主机的用户(默认是root)ansible.cfg中可配置
-k：指定远程主机的密码
-s：以sudo方式运行
-U：sudo到那个用户(默认是root)
-f：指定多少个进程并发处理(默认是5)
--private-key=/path：指定私钥路径
-T：ssh连接超时时间(默认(10s)
-t：日志输出到该目录
-v：显示详细信息
```




## Host Inventory

　　inventory文件用来定义需要被管理的主机清单，默认位置在`/etc/ansible/hosts`，不过不在默认位置，可以使用-i选项来指定。被管理的机器可以通过其IP或域名指定。每个中括号里代表一个分组，其下的机器列表都归属于这个分组，直到出现下一个中括号为止。通常我们按组来执行任务，同一组受控服务器应用相同的配置。一台服务器也可以归属到多个组，以完成多个功能角色的配置。低耦合、模块化，非常灵活！


### 主机清单配置

``` shell
[root@centos80 ~]# vim /etc/ansible/hosts

# 指定IP地址并且支持通配;
[web01]
192.168.56.80
192.168.56.[1-9]
 
# 指定IP加端口;
[web02]
192.168.56.81:5252
 
# 指定域名,必须可以解析;
[web03]
www.example.com
192.168.56.83

# 组嵌套,当执行组[weball]时就会执行它的所有子组但是子组可以独立执行;
[weball:children]
[web01]
[web02]
[web03]
```


### 查看所有配置受控节点

　　使用`ansible --list-hosts` 可以查看所有受控节点

``` shell
[root@centos80 ansible]# ansible all --list-hosts
  hosts (3):
    192.168.56.81
    192.168.56.82
    192.168.56.83
[root@centos80 ansible]# ansible webserver --list-hosts
  hosts (2):
    192.168.56.81
    192.168.56.82
[root@centos80 ansible]# ansible dbserver --list-hosts
  hosts (1):
    192.168.56.83
```

### 测试连通性



``` shell
ansible all -m ping --list-hosts

ansible all -m ping -C

[root@centos80 ansible]# ansible all -m ping
192.168.56.81 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
192.168.56.82 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
192.168.56.83 | SUCCESS => {
    "changed": false, 
    "ping": "pong"
}
```

### 常用参数


　　另外可以通过设置ansible_ssh_user来用指定用户在“受控节点”上执行任务，还可以通过设置ansible_ssh_host来指定不同的主机或域名，SSH对应的端口也可以通过ansible_ssh_port来修改，同时你还能使用特定的密钥登录。


**常用参数**

``` shell
ansible_connection=ssh                #指定连接类型,可以使local、ssh、paramiko;
ansible_ssh_user=root                 #用于指定远程主机的账号;
ansible_ssh_pass=password             #指定连接到主机的密码连-k都省了;
ansible_ssh_port=23                   #用于指定远程主机SSH端口;
ansible_ssh_privare_key_file=/PATH    #用于指定key文件;
aost_key_checking=false               #当第一次连接远程主机,跳过yes/no环节;
ansible_shell_type                    #指定目标系统的shell(默认为sh);
ansible_python_interpreter=/          #指定Python解释器路径(默认/USR/BIN/PYTHON);
```

示范

``` shell
[web01]
192.168.56.81 ansible_ssh_user=root
192.168.56.82 ansible_ssh_pass=password
```


### 主机变量和组变量

　　在inventory除了定义基本的参数外，还可以定义一些变量，以便在playbook中使用

``` shell
[web01]
192.168.56.81 http_prot=80 maxRequest=1000
192.168.56.82 http_prot=8080 maxRequest=1000
```

　　组也可以定义变量，复制给组内所有主机，这里的`vars`是固定格式

``` shell
[weball:vars]
http_port = 80
maxRequest = 100
```


## ansible参数说明

　　可能有些场景下，执行配置管理需要用root权限。但由于安全原因，可能会限制root使用SSH登录。比如：Ubuntu系统默认就不能使用 root 直接SSH登录系统。Ansible设计时也考虑到此类场景，这种情况下我们只需要告诉Ansible使用sudo的方式执行那些需要 root 权限的配置任务。前提条件是执行Ansible任务的用户需要有sudo的权限。要设定Ansible使用sudo，执行Ansible的任务的用户必须要有sudo权限。可以通过修改/etc/sudoers文件或visudo命令来完成，或者直接使用现在符合条件的用户。参数-sudo即告诉Ansible使用sudo来运行任务，如果sudo需要密码，则需要添加-k参数，或者在配置文件`/etc/ansible/ansible.cfg`中添加`ask_sudo_pass`的属性。

　　Ansible使用键值方式接受参数，即传统的KV方式（key=value）。每次执行任务后，将以JSON格式返回结果。它可以解析复杂的参数，或者用playbooks方式（后面会讲解）。Ansible返回会明确指明运行是否成功、是否有变动以及失败时的错信息。

　　通常都用`playbooks`的方式来执行Ansible任务，少数情况下使用命令行模式运行。过去，我们用Ansible的ping模块来检查“受控节点”是否正常受控。而实际上，ping模块仅仅执行了Ansible最核心的功能并检查了网络联通性，并未做其它实际性动作。

　　进而产生了`setup`模块，它不仅可以反馈“受控节点”的可用性，还会收集一些系统信息以供其它模块使用。Setup 模块定义了一系列的采集指令，比如：内核版本、机器名、IP地址等等，并将这些信息保存在内置变量，其它模块再执行任务时可以直接引用或用于判断条件等。


