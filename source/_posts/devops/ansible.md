---
title : ansible第一篇：介绍和安装
date : 2017-11-17 19:00:00
author : imkindu
categories : devops
tags :
- ansible
- 自动化运维
---

　　Ansible是一个简单的自动化运维管理工具，可以用来自动化部署应用、配置、编排 task(持续交付、无宕机更新等)， Ansible只需要在一台普通的服务器上运行即可，不需要在被管控的服务器上安装客户端。因为它是基于SSH的，Linux服务器离不开SSH，所以Ansible不需要为配置工作添加额外的支持。 你可以通过命令行来使用Ansible，运行Ansible的服务器这里俗称“管理节点”；通过Ansible进行管理的服务器俗称“受控节点”。


<!--more-->


<div align="center">
![](/images/ansible/logo.png)
</div>

## ansible特性

　　基于Python语言实现，其巧妙地通过SSH进行管理节点，被管理端无需Agent，使用起来非常方便。Ansible巧妙地利用了现有的组件进行组装称一个完整的系统，如利用：

- `Paramiko组件`（Python的SSH连接库）

- `PyYAML组件`（Python的YAML解析器库）

- `Jinja2组件`（Python的模板引擎库）

　　受控节点如果是Python 2.4 或 Python 2.5 ，则需额外安装Simplejson模块。到Python的2.6或以上版本，就则内置了Simplejson模块，不需要额外安装任何其它依赖。值得欣慰的是，目前主流的服务器上内置的Python版本绝多数都是Python 2.6以上版本。


## ansible架构

　　ansible在管理节点通过ansible模块利用ssh协议，将各命令推送到个被管理端执行。所以，管理端能管理各个节点的前提是：能通过ssh免密钥登录各被管理节点。

<div align="center">
![](/images/ansible/001.png)
</div>


`Host Inventory`：定义ansible管理的主机，可以进行分组管理。

`Core Modules` ：Ansible核心模块，ansible中模块就是用来指定对远程主机具体的操作，比如执行命令模块command、创建文件模块file等（ansible自带的模块）。

`Custom Modules` ：自定义模块，如果核心模块不足以完成某种功能就可以使用任何语言自定义模块。

`Connection Plugins`：连接插件是Ansible用来连接被管理端的一种方式，比如客户端运行了SSH服务，Ansible利用SSH服务跟客户端进行通信，Ansible是通过Python编写的，而Python有一个模块paramiko支持并行连接SSH。Ansible事实上就是使用paramiko进行连接被管理端。但是它还支持其他的连接方法需要插件（如zeroMQ就是C/S模式工作）。

`Plugins`：第三方插件支持，如email、logging模块，只要有Python编写能力就可以自行编写插件。

`Playbooks`：剧本或编辑是Ansible的任务配置文件、将多个任务定义在剧本中由Ansible自动执行，Playbooks最大的好处就是支持幂等，也就是说相同的任务不管执行多少次其结果是不变的，带来的好处也就是持续可维护性。





## ansible安装


　　ansible是基于Python语言研发的，所有可以使用pip工具安装最新版本，当然，epel源也有rpm包，可以使用Yum安装。

### yum安装ansible

　　使用YUM安装Ansible时需要配置epel源才行，能帮你自动解决软件包的依赖关系。

``` shell
$ rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
$ yum install ansible
```

``` shell
[root@centos80 ~]# yum install ansible
================================================================================================================================
 Package                                                           Arch                                 Version                 
================================================================================================================================
Installing:
 ansible                                                           noarch                               2.2.1.0-1.el7           
Installing for dependencies:
 PyYAML                                                            x86_64                               3.10-11.el7             
 libtomcrypt                                                       x86_64                               1.17-23.el7             
 libtommath                                                        x86_64                               0.42.0-4.el7            
 libyaml                                                           x86_64                               0.1.4-11.el7_0          
 python-babel                                                      noarch                               0.9.6-8.el7             
 python-backports                                                  x86_64                               1.0-8.el7               
 python-backports-ssl_match_hostname                               noarch                               3.4.0.2-4.el7           
 python-httplib2                                                   noarch                               0.7.7-3.el7             
 python-jinja2                                                     noarch                               2.7.2-2.el7             
 python-keyczar                                                    noarch                               0.71c-2.el7             
 python-markupsafe                                                 x86_64                               0.11-10.el7             
 python-setuptools                                                 noarch                               0.9.8-4.el7             
 python2-crypto                                                    x86_64                               2.6.1-13.el7            
 python2-ecdsa                                                     noarch                               0.13-4.el7              
 python2-paramiko               ssh连接各主机                       noarch                               1.16.1-2.el7            
 python2-pyasn1                                                    noarch                               0.1.9-7.el7             
 sshpass                                                           x86_64                               1.06-1.el7              

Transaction Summary
================================================================================================================================
Install  1 Package (+17 Dependent packages)
```


### pip安装ansible

　　如果使用pip安装Ansible。升级操作系统时，并不会同时升级Ansible。另外，升级操作系统有可能损坏Ansible环境，毕竟它依赖Python。Pip的安装指令为：
``` shell
$ rpm -ivh http://dl.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
$ yum install ansible
```


## ansible文件说明

我这里使用YUM安装的，查看一下Ansible相关文件。

``` shell
$ rpm -ql ansible | more
```

``` shell
# 主配置文件
/etc/ansible/ansible.cfg
 
# 默认定义主机清单文件
/etc/ansible/hosts
 
# 用来编排Playbook
/etc/ansible/roles
 
# 执行命令的程序
/usr/bin/ansible
/usr/bin/ansible-console
 
# Ansible模块使用帮助命令,其中使用-l可以查看ansible自带的所有模块
/usr/bin/ansible-doc
/usr/bin/ansible-galaxy
 
# 用来执行playbook的程序
/usr/bin/ansible-playbook
/usr/bin/ansible-pull
/usr/bin/ansible-vault
```

