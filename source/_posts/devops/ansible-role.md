---
title : ansible第六篇：roles角色
date : 2017-11-18 09:00:00
author : imkindu
categories : 
- devops
tags:
- ansible
- roles
---

　　playbook roles角色这个词一直不知道怎么解释，这么说吧，我们常用的lnmp环境，可以分为nginx管理、mysql管理、php管理等，如果我们使用playbook写成一个文件，这个文件会很大，但是不方便组织，我们可以分组，将其按大组分类，再细分为具体的小任务。

<!--more-->


## include

　　如果需要将一个大文件拆分为各个小文件，我们经常使用的就是`Include`，这是原先ansible拆分文件的做法，举个栗子。

一个 task include file 由一个普通的 task 列表所组成，像这样:

``` shell
---
# possibly saved as tasks/foo.yml

- name: placeholder foo
  command: /bin/foo

- name: placeholder bar
  command: /bin/bar
```

Include 指令看起来像下面这样，在一个 playbook 中，Include 指令可以跟普通的 task 混合在一起使用:

``` shell
tasks:
  - include: tasks/foo.yml
```



## roles

　　roles这个词：一个分类，将mysql、php等分为各自的大组，在各个角色内定义具体的小任务，方便管理，另一方面，类似于php的类的自动加载，roles基于一个已知的文件结构，可以自动去加载某些vars_files、tasks、handlers等。

　　一个项目的结构如下：

``` shell
site.yml
webservers.yml
fooservers.yml
roles/
   common/              # 一个角色包含了一个playbook的基本参数
     files/
     templates/         # 模板
     tasks/             # 任务
     handlers/          # 触发任务
     vars/              # 变量
     defaults/
     meta/
   webservers/
     files/
     templates/
     tasks/
     handlers/
     vars/
     defaults/
     meta/
```

**roles内各目录含义解释**

`files`：用来存放由copy模块或script模块调用的文件。
`templates`：用来存放jinjia2模板，template模块会自动在此目录中寻找jinjia2模板文件。
`tasks`：此目录应当包含一个main.yml文件，用于定义此角色的任务列表，此文件可以使用include包含其它的位于此目录的task文件。
`handlers`：此目录应当包含一个main.yml文件，用于定义此角色中触发条件时执行的动作。
`vars`：此目录应当包含一个main.yml文件，用于定义此角色用到的变量。
`defaults`：此目录应当包含一个main.yml文件，用于为当前角色设定默认变量。
`meta`：此目录应当包含一个main.yml文件，用于定义此角色的特殊设定及其依赖关系。


``` shell
---
- hosts: webservers
  roles:                    # 这里定义了使用哪些角色
     - common
     - webservers
```



这个 playbook 为一个角色 ‘xxx’ 指定了如下的行为：

> 如果 roles/xxx/tasks/main.yml 存在, 其中列出的 tasks 将被添加到 play 中
如果 roles/xxx/handlers/main.yml 存在, 其中列出的 handlers 将被添加到 play 中
如果 roles/xxx/vars/main.yml 存在, 其中列出的 variables 将被添加到 play 中
如果 roles/xxx/meta/main.yml 存在, 其中列出的 “角色依赖” 将被添加到 roles 列表中 (1.3 and later)
所有 copy tasks 可以引用 roles/xxx/files/ 中的文件，不需要指明文件的路径。
所有 script tasks 可以引用 roles/xxx/files/ 中的脚本，不需要指明文件的路径。
所有 template tasks 可以引用 roles/xxx/templates/ 中的文件，不需要指明文件的路径。
所有 include tasks 可以引用 roles/xxx/tasks/ 中的文件，不需要指明文件的路径。


## lamp场景分析


　　就算我们平时使用playbook时也应该使用roles来进行编排playbook，这样看上去比较清晰明了。但是roles还有更重要的任务那就是减少代码的重复性，这一点在我理解就是相当于模块化，把所有的功能都做成模块，谁需要用时调用即可。roles也是这样的概念，把各个角色独立编排，当需要使用一个或多个roles时只需要在play中指定即可。

<div align="center">
![](/images/ansible/23.png)
</div>

　　如上图安装一个LAMP环境，但是各自运行在独立的服务器上，并且它们都有一个公共需要的Common（用来定义一些系统初始化的任务）。这么一个场景，使用roles就非常好了，把PHP、Apache、MySQL分别独立成三个roles，然后把Common独立为一个roles。当需要安装服务时，只需要在各自的play中指定对应的roles即可，需要安装什么服务就调用什么roles。


## 创建roles步骤


1、创建以roles命令的目录

``` shell
[root@centos80 ansible]# mkdir /etc/ansible/roles
```

2、创建全局变量目录

``` shell
[root@centos80 ansible]# mkdir /etc/ansible/group_vars
[root@centos80 ansible]# touch /etc/ansible/group_vars/all      # 变量文件
```

3、定义角色

``` shell
[root@centos80 ansible]# mkdir /etc/ansible/roles/redis
```

4、位每个角色目录分别创建files、handlers、tasks、templates、meta、defaults和vars等目录，目录可以为空，但不能不创建


``` shell
[root@centos80 ansible]# mkdir /etc/ansible/roles/redis/{files,templates,tasks,handlers,vars,defaults,meta} -p
```

5、为每个角色目录下创建main.yml入口文件，通过此文件自动加载

``` redis
[root@centos80 ansible]# touch /etc/ansible/roles/redis/{defaults,vars,tasks,meta,handlers}/main.yml
```

``` shell
[root@centos80 tasks]# cat /etc/ansible/roles/redis/tasks/main.yml 
- name: restart redis
  tags: restart redis
  service: name=redis state=restarted
```



6、在playbook中调用角色

　　如果只需要执行playbook中某一个roles 某一个tags时，可以使用 -t 指定此roles 并使用tags

``` shell
[root@centos80 ansible]# cat roleredis.yml 
- hosts: 192.168.56.81
  remote_user: root
  roles:
  - { role: redis, tags: "restart redis" }      # 指明使用redis角色某个单独标签
  - php                                         # 指明使用php角色整个任务
```

