---
title : ansible第五篇：playbook
date : 2017-11-17 22:00:00
author : imkindu
categories :
- devops
- ansible
tags : 
- playbook
- ansible
---

　　前面使用ansible每次都是命令行指定使用什么模块，使用什么参数，只是一次性的，如果需要执行多次，还是要写入文件，方便多次执行。这就是playbook.

　　Playbook是Ansible的配置，部署，编排语言。他们可以被描述为一个需要希望远程主机执行命令的方案，或者一组IT程序运行的命令集合。

<!--more-->


## 关于Playbooks

Playbook是一种与adhoc任务执行模式完全不同的方式，而且特别强大。

简单地说，Playbooks是一个非常简单的配置管理和多机器部署系统的基础，不像任何已经存在的那样，而且非常适合部署复杂的应用程序。

Playbook可以声明配置，但它们也可以协调任何手动订购过程的步骤，即使不同的步骤必须在特定订单的机器组之间来回弹起。 他们可以同步或异步地启动任务。

虽然您可以运行主/usr/bin/ansible程序来进行自组织任务，但是Playbook更有可能被保留在源代码管理中，并用于推出配置或确保远程系统的配置符合规范。


## ansible-playbook命令

　　使用Playbook时通过`ansible-playbook`命令使用，它的参数和ansible命令类似，如参数-k(–ask-pass) 和-K (–ask-sudo) 来询问ssh密码和sudo密码，-u指定用户，这些指令也可以通过规定的单元写在playbook里。ansible-playbook的简单使用方法：


> ansible-playbook /etc/ansible/roles/redis.yml

　　可以使用 `-C` 先测试一遍

``` shell
[root@centos80 ansible]# ansible-playbook --help
Usage: ansible-playbook playbook.yml

Options:
  --ask-vault-pass      ask for vault password
  -C, --check           don't make any changes; instead, try to predict some
                        of the changes that may occur
  -D, --diff            when changing (small) files and templates, show the
                        differences in those files; works great with --check
  -e EXTRA_VARS, --extra-vars=EXTRA_VARS
                        set additional variables as key=value or YAML/JSON
  --flush-cache         clear the fact cache
  --force-handlers      run handlers even if a task fails
  -f FORKS, --forks=FORKS
                        specify number of parallel processes to use
                        (default=5)
  -h, --help            show this help message and exit
  -i INVENTORY, --inventory-file=INVENTORY
                        specify inventory host path
                        (default=/etc/ansible/hosts) or comma separated host
                        list.
  -l SUBSET, --limit=SUBSET
                        further limit selected hosts to an additional pattern
  --list-hosts          outputs a list of matching hosts; does not execute
                        anything else
  --list-tags           list all available tags
  --list-tasks          list all tasks that would be executed
  -M MODULE_PATH, --module-path=MODULE_PATH
                        specify path(s) to module library (default=None)
  --new-vault-password-file=NEW_VAULT_PASSWORD_FILE
                        new vault password file for rekey
  --output=OUTPUT_FILE  output file name for encrypt or decrypt; use - for
                        stdout
  --skip-tags=SKIP_TAGS
                        only run plays and tasks whose tags do not match these
                        values
  --start-at-task=START_AT_TASK
                        start the playbook at the task matching this name
  --step                one-step-at-a-time: confirm each task before running
  --syntax-check        perform a syntax check on the playbook, but do not
                        execute it
  -t TAGS, --tags=TAGS  only run plays and tasks tagged with these values
  --vault-password-file=VAULT_PASSWORD_FILE
                        vault password file
  -v, --verbose         verbose mode (-vvv for more, -vvvv to enable
                        connection debugging)
  --version             show program's version number and exit

  Connection Options:
    control as whom and how to connect to hosts

    -k, --ask-pass      ask for connection password
    --private-key=PRIVATE_KEY_FILE, --key-file=PRIVATE_KEY_FILE
                        use this file to authenticate the connection
    -u REMOTE_USER, --user=REMOTE_USER
                        connect as this user (default=None)
    -c CONNECTION, --connection=CONNECTION
                        connection type to use (default=smart)
    -T TIMEOUT, --timeout=TIMEOUT
                        override the connection timeout in seconds
                        (default=10)
    --ssh-common-args=SSH_COMMON_ARGS
                        specify common arguments to pass to sftp/scp/ssh (e.g.
                        ProxyCommand)
    --sftp-extra-args=SFTP_EXTRA_ARGS
                        specify extra arguments to pass to sftp only (e.g. -f,
                        -l)
    --scp-extra-args=SCP_EXTRA_ARGS
                        specify extra arguments to pass to scp only (e.g. -l)
    --ssh-extra-args=SSH_EXTRA_ARGS
                        specify extra arguments to pass to ssh only (e.g. -R)

  Privilege Escalation Options:
    control how and which user you become as on target hosts

    -s, --sudo          run operations with sudo (nopasswd) (deprecated, use
                        become)
    -U SUDO_USER, --sudo-user=SUDO_USER
                        desired sudo user (default=root) (deprecated, use
                        become)
    -S, --su            run operations with su (deprecated, use become)
    -R SU_USER, --su-user=SU_USER
                        run operations with su as this user (default=root)
                        (deprecated, use become)
    -b, --become        run operations with become (does not imply password
                        prompting)
    --become-method=BECOME_METHOD
                        privilege escalation method to use (default=sudo),
                        valid choices: [ sudo | su | pbrun | pfexec | doas |
                        dzdo | ksu ]
    --become-user=BECOME_USER
                        run operations as this user (default=root)
    --ask-sudo-pass     ask for sudo password (deprecated, use become)
    --ask-su-pass       ask for su password (deprecated, use become)
    -K, --ask-become-pass
                        ask for privilege escalation password
```





## playbook组成结构

``` text
Inventory
Modules
Ad Hot Commands
Playbooks
  Variables      #变量元素,可传递给Tasks/Templates使用;
  Tasks          #任务元素,即调用模块完成任务;
  Templates      #模板元素,可根据变量动态生成配置文件;
  Hadlers        #处理器元素,通常指在某事件满足时触发的操作;
  Roles          #角色元素
```


``` text
---                                 # YAML语法注释，类似shell的#
- hosts: webservers                 # 告诉ansible具体要在哪些主机上运行
  vars:                             # 变量
    http_port: 80
    max_clients: 200
  remote_user: root                 # 远程连接用户
  tasks:                            # 具体任务
  - name: ensure apache is at the latest version
    yum: pkg=httpd state=latest
  - name: write the apache config file
    template: src=/srv/httpd.j2 dest=/etc/httpd.conf
    notify:
    - restart apache
  - name: ensure apache is running
    service: name=httpd state=started
  handlers:
    - name: restart apache
      service: name=httpd state=restarted
```


　　简单示例：利用ansible-playbook停止redis服务

``` shell
[root@centos80 ansible]# cat redis.yml 
---
- hosts: 192.168.56.81
  vars:
    port: 6379
    name: redis
  tasks:
  - name: stop {{ name }}
    service: name={{ name }} state=stopped
```

<div align="center">
![](/images/ansible/03.png)
</div>

## 提示技巧

<div align="center">
![](/images/ansible/03.png)
</div>

- `ok`：表示执行成功但没有做过任何变动的任务。

- `changed`：表示执行成功但做过变动的任务。

- `unreachable`：表示无法到达的任务。

- `failed`：表示失败的任务


## 主机hosts和用户users

　　Playbook中的每一个paly的目的都是为了让某个或某些主机以某个指定的用户的身份执行任务，hosts用于指定要执行任务的主机，其可以是一个或多个由冒号分隔主机组，这和前面章节ansible命令提到的的hosts使用一样的语法。remote_user则用于指定远程主机上的执行任务的户。

<div align="center">
![](/images/ansible/04.png)
</div>

　　如果你需要在使用sudo时指定密码，可在运行ansible-playbook命令时加上选项 `--ask-sudo-pass(-K)`。如果使用sudo时，playbook疑似被挂起，可能是在sudo prompt处被卡住，这时可执行Control-C杀死卡住的任务，再重新运行一次。


## task列表 


　　Play的主体部分是task列表，task列表中的各任务按次序逐个在hosts中指定的主机上执行，即在所有主机上完成第一个任务后再开始第二个任务。在运行playbook 时（从上到下执行），如果一个host执行task失败，这个host将会从整个playbook的rotation中移除，如果发生执行失败的情况，请修正playbook 中的错误，然后重新执行即可。

　　Task的目的是使用指定的参数执行模块，而在模块参数中可以使用变量，模块执行时幂等的，这意味着多次执行是安全的，因为其结果一致。

　　对于command module和shell module，重复执行playbook，实际上是重复运行同样的命令。如果执行的命令类似于‘chmod’ 或者‘setsebool’这种命令，这没有任何问题，也可以使用一个叫做‘creates’的flag使得这两个module变得具有”幂等”特性 （不是必要的）。

　　每一个task必须有一个名称name，这样在运行playbook时，从其输出的任务执行信息中可以很好的辨别出是属于哪一个task的。如果没有定义name，‘action’的值将会用作输出信息中标记特定的task。

　　定义一个task，以前有一种格式: `action: module options` （可能在一些老的playbooks中还能见到），现在推荐使用更常见的格式:`module: options`，本文档使用的就是这种格式。

<div align="center">
![](/images/ansible/05.png)
</div>


　　在使用command和shell模块时，我们需要关心返回码信息，如果有一条命令，它的成功执行的返回码不是0（ansible可能会终止运行），我们可以使用如下方式替代，强制返回成功：

<div align="center">
![](/images/ansible/06.png)
</div>

　　或者使用ignore_error来忽略错误信息：

<div align="center">
![](/images/ansible/07.png)
</div>

## 变量variables

　　在playbook中可以直接定义变量，也可以在inventory中添加变量

<div align="center">
![](/images/ansible/10.png)
</div>

　　其中vars是固定格式，而package就是变量名，httpd就是变量值。

　　但在定义变量名时有一些规范，也就是合法的变量名。在使用变量之前最好先知道什么是合法的变量名，变量名可以为字母，数字以及下划线。变量始终应该以字母开头，“foo_port”是个合法的变量名。”foo5”也是， “foo-port”， “foo port”，“foo.port” 和“12”则不是合法的变量名。

**变量使用**


　　在task或template中引用变量都是使用双花括号中间引入变量名即可，如 `\{\{ VAR_NAME \}\}`。

<div align="center">
![](/images/ansible/11.png)
</div>

**setup模块**


　　Ansible中有一个setup模块，就是用来获取客户端所有信息的，如内存、CPU、IP、主机名、磁盘等等信息，这些信息都是key-value格式的。每当我们执行playbook时首先会需要执行setup模块。

<div align="center">
![](/images/ansible/12.png)
</div>

　　当我执行playbook时，其实是可以在task或template中去引用setup模块输出的变量。有哪些变量可用，你可以执行ansible命令看看输出结果：

<div align="center">
![](/images/ansible/13.png)
</div>



## handlers



　　Handlers其实挺好理解的，就是在发生改变时执行提前定义好的操作。

　　模块具有”幂等”性，所以当远端系统被人改动时，可以重放playbook达到恢复的目的。playbook本身可以识别这种改动，并且有一个基本的event system（事件系统），可以响应这种改动。（当发生改动时）’notify’actions会在playbook的每一个task结束时被触发，而且即使有多个不同的task通知改动的发生，’notify’ actions只会被触发一次。

``` shell
[root@centos80 ansible]# cat redis.yml 
---
- hosts: 192.168.56.81
  vars:
    port: 6379
    name: redis
  tasks:
  - name: install {{ name }}
    yum : name={{ name }} state=latest
  - name: redis.conf
    copy: src=/app/ansible/redis.conf dest=/etc/redis.conf
    notify: 
      - restart redis                   # 定义handlers,这里的name和下面的handlers要相同
  - name: start {{ name }}
    service: name={{ name }} state=started
  handlers:
    - name: restart {{ name }}
      service: name={{ name }} state=restarted
```

　　Handlers也是一些task的列表，通过名字来引用，它们和一般的task并没有什么区别。Handlers是由通知者进行notify，如果没有被 notify，handlers不会执行。不管有多少个通知者进行了notify，等到play中的所有task执行完成之后，handlers也只会被执行一次。

**注意**：notify定义的restart httpd必须与handlers定义的name名称相同，不然就会报：not found in any of the known handlers

<div align="center">
![](/images/ansible/14.jpg)
</div>


## 条件测试when


如果需要根据变量、facts（setup）或此前任务的执行结果来作为某task执行与否的前提时要用到条件测试。在Playbook中条件测试使用when子句。在task后添加when子句即可使用条件测试：when子句支持jinjia2表达式或语法

<div align="center">
![](/images/ansible/15.png)
</div>

如果是ubuntu系统，就进行关机。


## 迭代循环with_items

　　有时需要重复执行多个一样的任务时，需要使用迭代机制。相当于一个for循环，执行多次，其使用格式为定义要迭代的内容为`item`变量，并通过`with_items`语句指明迭代的元素列表。

<div align="center">
![](/images/ansible/20.png)
</div>

<div align="center">
![](/images/ansible/21.png)
</div>


## tags标签

　　在一个playbook中，我们可以定义多个task任务，但是如果我们只希望执行其中一个，不要按顺序执行，这时就需要使用tags标签指明play那个任务了。


``` shell
[root@centos80 ansible]# cat redis.yml 
---
- hosts: 192.168.56.81
  vars:
    port: 6379
    name: redis
  tasks:
  - name: install {{ name }}
    yum : name={{ name }} state=latest
  - name: redis.conf
    copy: src=/app/ansible/redis.conf dest=/etc/redis.conf
    tags:
    - redis.conf                                # 打上一个标签
    notify: 
      - restart redis
  - name: start {{ name }}
    service: name={{ name }} state=started
  handlers:
    - name: restart {{ name }}
      service: name={{ name }} state=restarted
```

　　使用`--tags`指明执行哪些标签任务

<div align="center">
![](/images/ansible/22.png)
</div>