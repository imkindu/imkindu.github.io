---
title : ansible第三篇：核心模块
date : 2017-11-17 20:50:00
author : imkindu
categories : devops
tags :
- ansible
---

　　ansible主要使用的就是各个模块，主要分为：核心模块和自定义模块。代码托管在 https://github.com/ansible

<!--more-->

　　核心模块主要分为：云模块、命令模块、数据库模块、文件模块、资产模块、消息模块、监控模块、网络模块、通知模块、包管理模块、源码控制模块、系统模块、单元模块、web设施模块、windows模块等。




## ansible-doc帮助文档

　　所有模块帮助文档，整个ansible的核心模块使用基本都需要查找此帮助。尤其是`-s`查看具体模块使用语法。

``` shell
[root@centos80 ~]# ansible-doc --help
Usage: ansible-doc [options] [module...]

Options:
  -h, --help            show this help message and exit
  -l, --list            List available modules
  -M MODULE_PATH, --module-path=MODULE_PATH
                        specify path(s) to module library (default=None)
  -s, --snippet         Show playbook snippet for specified module(s)
  -v, --verbose         verbose mode (-vvv for more, -vvvv to enable
                        connection debugging)
  --version             show program's version number and exit
```


## 模块列表

　　模块不要太多，主要语法是python开发的模块，所以可以直接查看python文件查看源码。

``` shell
ansible-doc --list            List available modules
[root@centos80 ~]# ansible-doc --list
a10_server                         Manage A10 Networks AX/SoftAX/Thunder/vThunder devices                                         
a10_service_group                  Manage A10 Networks devices' service groups                                                    
a10_virtual_server                 Manage A10 Networks devices' virtual servers                                                   
acl                                Sets and retrieves file ACL information.                                                       
add_host                           add a host (and alternatively a group) to the ansible-playbook in-memory inventory             
airbrake_deployment                Notify airbrake about app deployments      
......
......
```


## 常用模块介绍


　　模块太多，先熟悉一下常用核心模块基本使用。

## ping 

　　测试主机是否是通的，用法比较简单，不涉及参数，只需指明测试哪些主机。

``` shell
[root@centos80 ~]# ansible-doc  -s ping
- name: Try to connect to host, verify a usable python and return `pong' on success.
  action: ping


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




## user

　　用户模块，主要用户用户管理

``` shell
[root@centos80 ~]# ansible-doc -s user
- name: Manage user accounts
  action: user
      append                 # If `yes', will only add groups, not set them to just the list in `groups'.
      comment                # Optionally sets the description (aka `GECOS') of user account.
      createhome             # Unless set to `no', a home directory will be made for the user when the account is created or if the
                               home directory does not exist.
      expires                # An expiry time for the user in epoch, it will be ignored on platforms that do not support this.
                               Currently supported on Linux and FreeBSD.
      force                  # When used with `state=absent', behavior is as with `userdel --force'.
      generate_ssh_key       # Whether to generate a SSH key for the user in question. This will *not* overwrite an existing SSH key.
      group                  # Optionally sets the user's primary group (takes a group name).
      groups                 # Puts the user in this comma-delimited list of groups. When set to the empty string ('groups='), the
                               user is removed from all groups except the primary group.
      home                   # Optionally set the user's home directory.
      login_class            # Optionally sets the user's login class for FreeBSD, OpenBSD and NetBSD systems.
      move_home              # If set to `yes' when used with `home=', attempt to move the user's home directory to the specified
                               directory if it isn't there already.
      name=                  # Name of the user to create, remove or modify.
      non_unique             # Optionally when used with the -u option, this option allows to change the user ID to a non-unique
                               value.
      password               # Optionally set the user's password to this crypted value.  See the user example in the github examples
                               directory for what this looks like in a playbook. See
                               http://docs.ansible.com/ansible/faq.html#how-do-i-generate-crypted-
                               passwords-for-the-user-module for details on various ways to generate
                               these password values. Note on Darwin system, this value has to be
                               cleartext. Beware of security issues.
      remove                 # When used with `state=absent', behavior is as with `userdel --remove'.
      seuser                 # Optionally sets the seuser type (user_u) on selinux enabled systems.
      shell                  # Optionally set the user's shell.
      skeleton               # Optionally set a home skeleton directory. Requires createhome option!
      ssh_key_bits           # Optionally specify number of bits in SSH key to create.
      ssh_key_comment        # Optionally define the comment for the SSH key.
      ssh_key_file           # Optionally specify the SSH key filename. If this is a relative filename then it will be relative to the
                               user's home directory.
      ssh_key_passphrase     # Set a passphrase for the SSH key.  If no passphrase is provided, the SSH key will default to having no
                               passphrase.
      ssh_key_type           # Optionally specify the type of SSH key to generate. Available SSH key types will depend on
                               implementation present on target host.
      state                  # Whether the account should exist or not, taking action if the state is different from what is stated.
      system                 # When creating an account, setting this to `yes' makes the user a system account.  This setting cannot
                               be changed on existing users.
      uid                    # Optionally sets the `UID' of the user.
      update_password        # `always' will update passwords if they differ.  `on_create' will only set the password for newly
                               created users.
```

常用参数

``` shell
home        #指定用户的家目录，需要与createhome配合使用;
groups      #指定用户的属组;
uid         #指定用的uid;
password    #指定用户的密码;
name        #指定用户名;
createhome  #是否创建家目录yes|no;
system      #是否为系统用户;
remove      #当state=absent时，remove=yes则表示连同家目录一起删除，等价于userdel -r;
state       #是创建还是删除;
shell       #指定用户的shell环境;
```
示例

``` shell
[root@centos80 ~]# ansible all -m user -a "name=dandan group=dandan shell=/bin/bash state=present"
192.168.56.81 | SUCCESS => {
    "changed": true, 
    "comment": "", 
    "createhome": true, 
    "group": 1000, 
    "home": "/home/dandan", 
    "name": "dandan", 
    "shell": "/bin/bash", 
    "state": "present", 
    "system": false, 
    "uid": 1000
}
192.168.56.82 | SUCCESS => {
    "changed": true, 
    "comment": "", 
    "createhome": true, 
    "group": 1000, 
    "home": "/home/dandan", 
    "name": "dandan", 
    "shell": "/bin/bash", 
    "state": "present", 
    "system": false, 
    "uid": 1000
}
```



## group

　　用户组管理

``` shell
[root@centos80 ~]# ansible-doc -s group
- name: Add or remove groups
  action: group
      gid                    # Optional `GID' to set for the group.
      name=                  # Name of the group to manage.
      state                  # Whether the group should be present or not on the remote host.
      system                 # If `yes', indicates that the group created is a system group.
```

添加用户组
``` shell
[root@centos80 ~]# ansible all -m group -a "name=dandan state=present system=no"
192.168.56.82 | SUCCESS => {
    "changed": true, 
    "gid": 1000, 
    "name": "dandan", 
    "state": "present", 
    "system": false
}
192.168.56.81 | SUCCESS => {
    "changed": true, 
    "gid": 1000, 
    "name": "dandan", 
    "state": "present", 
    "system": false
}
```

删除用户组

``` shell
[root@centos80 ~]# ansible all -m group -a "name=dandan state=absent "
```





## copy

　　复制文件管理

``` shell
[root@centos80 ansible]# ansible-doc -s copy
- name: Copies files to remote locations.
  action: copy
      backup                 # Create a backup file including the timestamp information so you can get the original file back if you somehow clobbered it incorrectly.
      content                # When used instead of 'src', sets the contents of a file directly to the specified value. This is for simple values, for anything complex or with
                               formatting please switch to the template module.
      dest=                  # Remote absolute path where the file should be copied to. If src is a directory, this must be a directory too.
      directory_mode         # When doing a recursive copy set the mode for the directories. If this is not set we will use the system defaults. The mode is only set on
                               directories which are newly created, and will not affect those that already existed.
      follow                 # This flag indicates that filesystem links, if they exist, should be followed.
      force                  # the default is `yes', which will replace the remote file when contents are different than the source. If `no', the file will only be transferred
                               if the destination does not exist.
      group                  # name of the group that should own the file/directory, as would be fed to `chown'
      mode                   # mode the file or directory should be. For those used to `/usr/bin/chmod' remember that modes are actually octal numbers (like 0644). Leaving off
                               the leading zero will likely have unexpected results. As of version 1.8, the mode may be specified as a symbolic
                               mode (for example, `u+rwx' or `u=rw,g=r,o=r').
      owner                  # name of the user that should own the file/directory, as would be fed to `chown'
      remote_src             # If False, it will search for src at originating/master machine, if True it will go to the remote/target machine for the src. Default is False.
                               Currently remote_src does not support recursive copying.
      selevel                # level part of the SELinux file context. This is the MLS/MCS attribute, sometimes known as the `range'. `_default' feature works as for `seuser'.
      serole                 # role part of SELinux file context, `_default' feature works as for `seuser'.
      setype                 # type part of SELinux file context, `_default' feature works as for `seuser'.
      seuser                 # user part of SELinux file context. Will default to system policy, if applicable. If set to `_default', it will use the `user' portion of the
                               policy if available
      src                    # Local path to a file to copy to the remote server; can be absolute or relative. If path is a directory, it is copied recursively. In this case, if
                               path ends with "/", only inside contents of that directory are copied to destination. Otherwise, if it does not end
                               with "/", the directory itself with all contents is copied. This behavior is similar to Rsync.
      unsafe_writes          # Normally this module uses atomic operations to prevent data corruption or inconsistent reads from the target files, sometimes systems are
                               configured or just broken in ways that prevent this. One example are docker mounted files, they cannot be updated
                               atomically and can only be done in an unsafe manner. This boolean option allows ansible to fall back to unsafe
                               methods of updating files for those cases in which you do not have any other choice. Be aware that this is subject
                               to race conditions and can lead to data corruption.
      validate               # The validation command to run before copying into place. The path to the file to validate is passed in via '%s' which must be present as in the
                               example below. The command is passed securely so shell features like expansion and pipes won't work.
```

拷贝hosts文件到目标主机

```
[root@centos80 ansible]# ansible webserver -m copy -a "src=hosts dest=/app/ owner=nobody mode=664"
192.168.56.82 | SUCCESS => {
    "changed": true, 
    "checksum": "49c073032d2e377dc43cf78fdc785a2c0c9f9afb", 
    "dest": "/app/hosts", 
    "gid": 0, 
    "group": "root", 
    "md5sum": "8a8ee6929aff9d2c04bdf4c9172a82f0", 
    "mode": "0664", 
    "owner": "nobody", 
    "size": 1085, 
    "src": "/root/.ansible/tmp/ansible-tmp-1510890819.67-105660654269079/source", 
    "state": "file", 
    "uid": 99
}
192.168.56.81 | SUCCESS => {
    "changed": true, 
    "checksum": "49c073032d2e377dc43cf78fdc785a2c0c9f9afb", 
    "dest": "/app/hosts", 
    "gid": 0, 
    "group": "root", 
    "md5sum": "8a8ee6929aff9d2c04bdf4c9172a82f0", 
    "mode": "0664", 
    "owner": "nobody", 
    "size": 1085, 
    "src": "/root/.ansible/tmp/ansible-tmp-1510890819.69-125724656278827/source", 
    "state": "file", 
    "uid": 99
}
```

``` shell
[root@centos81 ~]# ll /app/
total 4
drwxr-xr-x 2 root   root    6 Nov 16 20:43 ansible
-rw-rw-r-- 1 nobody root 1085 Nov 17 11:53 hosts        # 拷贝过来hosts文件
drwxr-xr-x 3 root   root   20 Nov 14 16:24 mysql
```





## fetch

　　从远程复制文件到本地，不能指定多个远程主机（多个主机如果都存在这个文件？？？怎么复制）

```shell
[root@centos80 ~]# ansible-doc -s fetch
- name: Fetches a file from remote nodes
  action: fetch
      dest=                  # A directory to save the file into. For example, if the `dest' directory is `/backup' a `src' file named
                               `/etc/profile' on host `host.example.com', would be saved into
                               `/backup/host.example.com/etc/profile'
      fail_on_missing        # When set to 'yes', the task will fail if the source file is missing.
      flat                   # Allows you to override the default behavior of appending hostname/path/to/file to the destination.  If
                               dest ends with '/', it will use the basename of the source file, similar
                               to the copy module. Obviously this is only handy if the filenames are
                               unique.
      src=                   # The file on the remote system to fetch. This `must' be a file, not a directory. Recursive fetching may
                               be supported in a later release.
      validate_checksum      # Verify that the source and destination checksums match after the files are fetched.
```


## file

　　创建文件  设置文件属性

``` shell
[root@centos80 ~]# ansible all -m file -a "path=/app/ansible state=directory"
192.168.56.81 | SUCCESS => {
    "changed": true, 
    "gid": 0, 
    "group": "root", 
    "mode": "0755", 
    "owner": "root", 
    "path": "/app/ansible", 
    "size": 6, 
    "state": "directory", 
    "uid": 0
}
192.168.56.82 | SUCCESS => {
    "changed": true, 
    "gid": 0, 
    "group": "root", 
    "mode": "0755", 
    "owner": "root", 
    "path": "/app/ansible", 
    "size": 6, 
    "state": "directory", 
    "uid": 0
}
```

但是不能创建空文件，如果要创建一个空文件，可以使用copy一个文件，content为空

``` shell
[root@centos80 ~]# ansible all -m file -a "path=/app/ansible.txt state=file"
192.168.56.81 | FAILED! => {
    "changed": false, 
    "failed": true,             # 创建失败
    "msg": "file (/app/ansible.txt) is absent, cannot continue", 
    "path": "/app/ansible.txt", 
    "state": "absent"
}
192.168.56.82 | FAILED! => {
    "changed": false, 
    "failed": true, 
    "msg": "file (/app/ansible.txt) is absent, cannot continue", 
    "path": "/app/ansible.txt", 
    "state": "absent"
}
```

## get_url

　　下载指定url文件到目标主机，主要用于从http、ftp、https服务器上下载文件

``` shell
[root@centos80 ~]# ansible-doc  -s get_url
Usage: ansible-doc [options] [module...]

Options:
url=                        #下载的URL;
dest=                       # 指明下载到什么位置
sha256sum                   #下载完成后进行sha256 check;
timeout                     #下载超时时间，默认10s;
url_password、url_username  #主要用于需要用户名密码进行验证的情况;
use_proxy                   #使用代理,代理需事先在环境变更中定义;
```

**示例**

``` shell
ansible test -m filesystem -a 'url=http://example.com/path/file.conf dest=/etc/foo.conf mode=0440'
```




## command

　　在远程主机执行命令

``` shell
[root@centos80 ~]# ansible-doc -s command
- name: Executes a command on a remote node
  action: command
      chdir                  # 切换目录执行  cd into this directory before running the command
      creates                # 指定文件或目录存在则不执行  a filename or (since 2.0) glob pattern, when it already exists, this step will *not* be run.
      executable             # change the shell used to execute the command. Should be an absolute path to the executable.
      free_form=             # the command module takes a free form command to run.  There is no parameter actually named 'free form'. See the examples!
      removes                # 如果文件不存在则不执行 a filename or (since 2.0) glob pattern, when it does not exist, this step will *not* be run.
      warn                   # if command warnings are on in ansible.cfg, do not warn about this particular line if set to no/false.
```


**远程执行命令**

``` shell
[root@centos80 ansible]# ansible all -m command -a hostname
192.168.56.81 | SUCCESS | rc=0 >>
centos81

192.168.56.83 | SUCCESS | rc=0 >>
centos83

192.168.56.82 | SUCCESS | rc=0 >>
centos82
```



**问题：**

这里使用管道修改密码，直接打印了出来，并未真正执行

``` shell
[root@centos80 ~]# ansible all -m command -a "echo dandan | passwd --stdin dandan"
192.168.56.81 | SUCCESS | rc=0 >>
dandan | passwd --stdin dandan

192.168.56.82 | SUCCESS | rc=0 >>
dandan | passwd --stdin dandan
```

## shell

　　上述问题使用shell模块即可，可是使用管道等shell命令特性

``` shell
[root@centos80 ~]# ansible all -m shell -a "echo dandan | passwd --stdin dandan"
192.168.56.81 | SUCCESS | rc=0 >>
Changing password for user dandan.
passwd: all authentication tokens updated successfully.

192.168.56.82 | SUCCESS | rc=0 >>
Changing password for user dandan.
passwd: all authentication tokens updated successfully.
```






## cron

　　计划任务管理

**常用参数**

``` shell
name          #该任务的描述;
backup        #对远程主机上的原任务计划内容修改之前做备份;
cron_file     #如果指定该选项，则用该文件替换远程主机上的cron.d目录下的用户的任务计划;
day           #日（1-31，，/2,……）;
hour          #小时（0-23，，/2，……）;
minute        #分钟（0-59，，/2，……）;
month         #月（1-12，，/2，……）;
weekday       #周（0-7，*，……）;
job           #要执行的任务,依赖于state=present;
special_time  #指定什么时候执行，参数：reboot(重启时),yearly(每年),annually,monthly,weekly,daily,hourly;
state         #确认该任务计划是创建还是删除;
user          #以哪个用户的身份执行;
```

**示例**


``` shell
$ ansible test -m cron -a 'name="a job for reboot" special_time=reboot job="/some/job.sh"'
$ ansible test -m cron -a 'name="yum autoupdate" minute=*/1 hour=* day=* month=* weekday=* user="root" job="date>>/tmp/1.txt"'
$ ansible test -m cron -a 'name="yum autoupdate" minute=1 hour=*/1 day=* month=* weekday=* user="root" job="date>>/tmp/1.txt"'
$ ansible test -m cron -a 'backup="True" name="test" minute="0" hour="5,2" job="ls -alh > /dev/null"'
$ ansilbe test -m cron -a 'cron_file=ansible_yum-autoupdate state=absent'
```
## yum


　　使用yum包管理器来管理软件包

**常用参数**

``` shell
name               #要进行操作的软件包的名字，也可以传递一个url或者一个本地的rpm包的路径;
config_file        #yum的配置文件;
disable_gpg_check  #关闭gpg_check;
disablerepo        #不启用某个源;
enablerepo         #启用某个源;
state              #用于描述安装包最终状态，<em>present/latest</em>用于安装包，<em>absent</em>用于卸载安装包;
```

**示例**

``` shell
# 安装最新版本的apache;
$ ansible test -m yum -a 'name=httpd state=latest'
 
# 移除apache;
$ ansible test -m yum -a 'name=httpd state=absent'
 
# 升级所有的软件包;
$ ansible test -m yum -a 'name=* state=latest '
 
# 安装整个Development tools相关的软件包;
$ ansible test -m yum -a 'name="@Development tools" state=present'
 
# 从本地仓库安装nginx;
$ ansible test -m yum -a 'name=/usr/local/src/nginx-release-centos-6-0.el6.ngx.noarch.rpm state=present'
 
# 从一个远程yum仓库安装nginx;
$ ansible test -m yum -a 'name=http://nginx.org/packages/centos/6/noarch/RPMS/nginx-release-centos-6-0.el6.ngx.noarch.rpm state=present'
```

## service

　　主要用于管理主机服务

**常用参数**


``` shell
name        #必选项,服务名称;
state       #对当前服务执行启动，停止、重启、重新加载等操作（started, stopped, restarted, reloaded）;
enabled     #是否开机启动yes|no;
pattern     #定义一个模式，如果通过status指令来查看服务的状态时，没有响应，就会通过ps指令在进程中根据该模式进行查找，如果匹配到，则认为该服务依然在运行;
runlevel    #运行级别;
arguments   #给命令行提供一些选项;
sleep       #如果执行了restarted，在则stop和start之间沉睡几秒钟;
``` 

**示例**

``` shell
$ ansible test -m service -a "name=httpd state=started enabled=yes"
$ ansible test -m service -a "name=foo pattern=/usr/bin/foo state=started"
$ ansible test -m service -a "name=network state=restarted args=eth0"
```

``` shell
[root@centos80 ~]# ansible all -m service -a "name=mariadb state=started"
192.168.56.82 | SUCCESS => {
    "changed": true, 
    "name": "mariadb", 
    "state": "started", 
    "status": {
        "ActiveEnterTimestamp": "Wed 2017-11-15 20:55:42 CST", 
        "ActiveEnterTimestampMonotonic": "2716911810", 
        "ActiveExitTimestamp": "Thu 2017-11-16 21:04:11 CST", 
        "ActiveExitTimestampMonotonic": "51948886270", 
        "ActiveState": "inactive", 
        "After": "systemd-journald.socket basic.target -.mount syslog.target network.target tmp.mount system.slice", 
        "AllowIsolate": "no", 
        "AssertResult": "yes", 
        "AssertTimestamp": "Wed 2017-11-15 20:55:39 CST", 
        "AssertTimestampMonotonic": "2713474837", 
        "Before": "shutdown.target multi-user.target", 
        "BlockIOAccounting": "no", 
        "BlockIOWeight": "18446744073709551615", 
        "CPUAccounting": "no", 
        "CPUQuotaPerSecUSec": "infinity", 
        "CPUSchedulingPolicy": "0", 
        "CPUSchedulingPriority": "0", 
        "CPUSchedulingResetOnFork": "no", 
        "CPUShares": "18446744073709551615", 
        "CanIsolate": "no", 
        "CanReload": "no", 
        "CanStart": "yes", 
        "CanStop": "yes", 
        "CapabilityBoundingSet": "18446744073709551615", 
        "ConditionResult": "yes", 
        "ConditionTimestamp": "Wed 2017-11-15 20:55:39 CST", 
        "ConditionTimestampMonotonic": "2713474836", 
        "Conflicts": "shutdown.target", 
        "ControlPID": "0", 
        "DefaultDependencies": "yes", 
        "Delegate": "no", 
        "Description": "MariaDB database server", 
        "DevicePolicy": "auto", 
        "ExecMainCode": "1", 
        "ExecMainExitTimestamp": "Thu 2017-11-16 21:04:12 CST", 
        "ExecMainExitTimestampMonotonic": "51949781779", 
        "ExecMainPID": "9883", 
        "ExecMainStartTimestamp": "Wed 2017-11-15 20:55:39 CST", 
        "ExecMainStartTimestampMonotonic": "2713725424", 
        "ExecMainStatus": "0", 
        "ExecStart": "{ path=/usr/bin/mysqld_safe ; argv[]=/usr/bin/mysqld_safe --basedir=/usr ; ignore_errors=no ; start_time=[n/a] ; stop_time=[n/a] ; pid=0 ; code=(null) ; status=0/0 }", 
        "ExecStartPost": "{ path=/usr/libexec/mariadb-wait-ready ; argv[]=/usr/libexec/mariadb-wait-ready $MAINPID ; ignore_errors=no ; start_time=[n/a] ; stop_time=[n/a] ; pid=0 ; code=(null) ; status=0/0 }", 
        "ExecStartPre": "{ path=/usr/libexec/mariadb-prepare-db-dir ; argv[]=/usr/libexec/mariadb-prepare-db-dir %n ; ignore_errors=no ; start_time=[n/a] ; stop_time=[n/a] ; pid=0 ; code=(null) ; status=0/0 }", 
        "FailureAction": "none", 
        "FileDescriptorStoreMax": "0", 
        "FragmentPath": "/usr/lib/systemd/system/mariadb.service", 
        "Group": "mysql", 
        "GuessMainPID": "yes", 
        "IOScheduling": "0", 
        "Id": "mariadb.service", 
        "IgnoreOnIsolate": "no", 
        "IgnoreOnSnapshot": "no", 
        "IgnoreSIGPIPE": "yes", 
        "InactiveEnterTimestamp": "Thu 2017-11-16 21:04:12 CST", 
        "InactiveEnterTimestampMonotonic": "51949781892", 
        "InactiveExitTimestamp": "Wed 2017-11-15 20:55:39 CST", 
        "InactiveExitTimestampMonotonic": "2713476218", 
        "JobTimeoutAction": "none", 
        "JobTimeoutUSec": "0", 
        "KillMode": "control-group", 
        "KillSignal": "15", 
        "LimitAS": "18446744073709551615", 
        "LimitCORE": "18446744073709551615", 
        "LimitCPU": "18446744073709551615", 
        "LimitDATA": "18446744073709551615", 
        "LimitFSIZE": "18446744073709551615", 
        "LimitLOCKS": "18446744073709551615", 
        "LimitMEMLOCK": "65536", 
        "LimitMSGQUEUE": "819200", 
        "LimitNICE": "0", 
        "LimitNOFILE": "4096", 
        "LimitNPROC": "5701", 
        "LimitRSS": "18446744073709551615", 
        "LimitRTPRIO": "0", 
        "LimitRTTIME": "18446744073709551615", 
        "LimitSIGPENDING": "5701", 
        "LimitSTACK": "18446744073709551615", 
        "LoadState": "loaded", 
        "MainPID": "0", 
        "MemoryAccounting": "no", 
        "MemoryCurrent": "18446744073709551615", 
        "MemoryLimit": "18446744073709551615", 
        "MountFlags": "0", 
        "Names": "mariadb.service", 
        "NeedDaemonReload": "no", 
        "Nice": "0", 
        "NoNewPrivileges": "no", 
        "NonBlocking": "no", 
        "NotifyAccess": "none", 
        "OOMScoreAdjust": "0", 
        "OnFailureJobMode": "replace", 
        "PermissionsStartOnly": "no", 
        "PrivateDevices": "no", 
        "PrivateNetwork": "no", 
        "PrivateTmp": "yes", 
        "ProtectHome": "no", 
        "ProtectSystem": "no", 
        "RefuseManualStart": "no", 
        "RefuseManualStop": "no", 
        "RemainAfterExit": "no", 
        "Requires": "basic.target -.mount", 
        "RequiresMountsFor": "/var/tmp", 
        "Restart": "no", 
        "RestartUSec": "100ms", 
        "Result": "success", 
        "RootDirectoryStartOnly": "no", 
        "RuntimeDirectoryMode": "0755", 
        "SameProcessGroup": "no", 
        "SecureBits": "0", 
        "SendSIGHUP": "no", 
        "SendSIGKILL": "yes", 
        "Slice": "system.slice", 
        "StandardError": "inherit", 
        "StandardInput": "null", 
        "StandardOutput": "journal", 
        "StartLimitAction": "none", 
        "StartLimitBurst": "5", 
        "StartLimitInterval": "10000000", 
        "StartupBlockIOWeight": "18446744073709551615", 
        "StartupCPUShares": "18446744073709551615", 
        "StatusErrno": "0", 
        "StopWhenUnneeded": "no", 
        "SubState": "dead", 
        "SyslogLevelPrefix": "yes", 
        "SyslogPriority": "30", 
        "SystemCallErrorNumber": "0", 
        "TTYReset": "no", 
        "TTYVHangup": "no", 
        "TTYVTDisallocate": "no", 
        "TimeoutStartUSec": "5min", 
        "TimeoutStopUSec": "5min", 
        "TimerSlackNSec": "50000", 
        "Transient": "no", 
        "Type": "simple", 
        "UMask": "0022", 
        "UnitFilePreset": "disabled", 
        "UnitFileState": "enabled", 
        "User": "mysql", 
        "WantedBy": "multi-user.target", 
        "Wants": "system.slice", 
        "WatchdogTimestampMonotonic": "0", 
        "WatchdogUSec": "0"
    }, 
    "warnings": []
}
```