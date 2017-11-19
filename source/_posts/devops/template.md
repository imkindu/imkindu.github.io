---
title : ansible第七篇：template模板
date : 2017-11-18 11:00:00
author : imkindu
categories : 
- devops
tags:
- ansible
- template
---


　　模板的定义个人这么解释吧：例如我们修改redis配置文件的maxmemory，在前面的修改过程中，我们都是修改好具体的值后，将此配置文件copy过去，直接替换，但是如果每个服务器的可使用内存大小不一致呢，这里我们就不能使用固定的值，需要在使用的文件中使用变量 `\{\{ ansible_memtotal_mb \}\}`，这个变量是setup模块查看环境时获取的，我们将`redis.conf`里面的`maxmemory = \{\{ ansible_memtotal_mb /2 \}\}mb`，这样就可以根据具体情况自动修改了。


<!--more-->

## template帮助


　　先来查看一下使用template有哪些参数

``` shell
- name: Templates a file out to a remote server.
  action: template
      backup                 # Create a backup file including the timestamp information so you can get the original file back if you
                               somehow clobbered it incorrectly.
      dest=                  # Location to render the template to on the remote machine.
      force                  # the default is `yes', which will replace the remote file when contents are different than the source.
                               If `no', the file will only be transferred if the destination does not
                               exist.
      group                  # name of the group that should own the file/directory, as would be fed to `chown'
      mode                   # mode the file or directory should be. For those used to `/usr/bin/chmod' remember that modes are
                               actually octal numbers (like 0644). Leaving off the leading zero will
                               likely have unexpected results. As of version 1.8, the mode may be
                               specified as a symbolic mode (for example, `u+rwx' or `u=rw,g=r,o=r').
      owner                  # name of the user that should own the file/directory, as would be fed to `chown'
      selevel                # level part of the SELinux file context. This is the MLS/MCS attribute, sometimes known as the `range'.
                               `_default' feature works as for `seuser'.
      serole                 # role part of SELinux file context, `_default' feature works as for `seuser'.
      setype                 # type part of SELinux file context, `_default' feature works as for `seuser'.
      seuser                 # user part of SELinux file context. Will default to system policy, if applicable. If set to `_default',
                               it will use the `user' portion of the policy if available
      src=                   # Path of a Jinja2 formatted template on the Ansible controller. This can be a relative or absolute path.
      unsafe_writes          # Normally this module uses atomic operations to prevent data corruption or inconsistent reads from the
                               target files, sometimes systems are configured or just broken in ways
                               that prevent this. One example are docker mounted files, they cannot be
                               updated atomically and can only be done in an unsafe manner. This
                               boolean option allows ansible to fall back to unsafe methods of updating
                               files for those cases in which you do not have any other choice. Be
                               aware that this is subject to race conditions and can lead to data
                               corruption.
      validate               # The validation command to run before copying into place. The path to the file to validate is passed in
                               via '%s' which must be present as in the example below. The command is
                               passed securely so shell features like expansion and pipes won't work.
```

主要的参数：

`src` ： 指定本地目标文件jinjia2模板文件位置
`dest` ： 指明远程主机文件所需要放置位置


## jinjia2

　　首先明确一点，ansible是python语言开发的，ansible里面的template使用的jinjia2模板引擎，所以模板文件后缀都为`j2`，类似于php的smarty

　　例如，准备一个redis.conf的模板文件

``` shell
cp /etc/redis.conf /etc/ansible/roles/redis/templates/redis.conf.j2
```


在模板中使用变量

``` text
maxmemory {{ ansible_memtotal_mb /2 }}mb
```


## 定义tasks

　　利用redis的roles角色，定义修改redis.conf配置文件并重启

``` shell
[root@centos80 redis]# cat /etc/ansible/redis/tasks/main.yml 
  - name: install {{ name }}
    yum : name={{ name }} state=latest
  - name: redis.conf
    template: src=redis.conf.j2 dest=/etc/redis.conf
    tags:
    - redis.conf
    notify: 
      - restart redis                       # 如果修改配置文件就重启服务
  - name: start {{ name }}
    service: name={{ name }} state=started
```

## 定义redis角色handlers


``` shell
[root@centos80 redis]# cat /etc/ansible/redis/handlers/main.yml 
- name: restart redis
  service: name=redis state=restarted
```

## 模板使用

　　template模板的使用同其他tags一样，只能具体参数即可

<div align="center">
![](/images/ansible/24.png)
</div>

## 检查

　　检查centos81上redis.conf最大内存变成了数值

``` shell
[root@centos81 etc]# grep "maxmemory" redis.conf
# according to the eviction policy selected (see maxmemory-policy).
# WARNING: If you have slaves attached to an instance with maxmemory on,
# limit for maxmemory so that there is some free RAM on the system for slave
# maxmemory <bytes>
maxmemory 722.0mb
```



