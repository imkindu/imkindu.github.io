---
title : MHA
date : 2017-11-17 09:00:00
categories : database
tags :
- mha
---

　　MHA是一款开源的mysql的高可用程序，它为mysql主从复制架构提供了automating master failover功能。MHA在监控到master节点故障时，会提升其中拥有最新数据的slave节点成为新的master节点，在此期间，MHA会通过与其他从节点获取额外的信息来避免不一致性的问题。MHA还提供了master节点的在线切换功能，即按需切换master/slave节点。


<!--more-->

## 原理

　　该软件由两部分组成：MHA Manager（管理节点）和MHA Node（数据节点）。MHA Manager可以单独部署在一台独立的机器上管理多个master-slave集群，也可以部署在一台slave节点上。MHA Node运行在每台MySQL服务器上，MHA Manager会定时探测集群中的master节点，当master出现故障时，它可以自动将最新数据的slave提升为新的master，然后将所有其他的slave重新指向新的master。整个故障转移过程对应用程序完全透明。

　　在MHA自动故障切换过程中，MHA试图从宕机的主服务器上保存二进制日志，最大程度的保证数据的不丢失，但这并不总是可行的。例如，如果主服务器硬件故障或无法通过ssh访问，MHA没法保存二进制日志，只进行故障转移而丢失了最新的数据。使用MySQL 5.5的半同步复制，可以大大降低数据丢失的风险。MHA可以与半同步复制结合起来。如果只有一个slave已经收到了最新的二进制日志，MHA可以将最新的二进制日志应用于其他所有的slave服务器上，因此可以保证所有节点的数据一致性。


## 构成

masterha_check_ssh              检查MHA的SSH配置状况
masterha_check_repl             检查MySQL复制状况
masterha_manger                 启动MHA
masterha_check_status           检测当前MHA运行状态
masterha_master_monitor         检测master是否宕机
masterha_master_switch          控制故障转移（自动或者手动）
masterha_conf_host              添加或删除配置的server信息


save_binary_logs                保存和复制master的二进制日志
apply_diff_relay_logs           识别差异的中继日志事件并将其差异的事件应用于其他的slave
filter_mysqlbinlog              去除不必要的ROLLBACK事件（MHA已不再使用这个工具）
purge_relay_logs                清除中继日志（不会阻塞SQL线程）


## ssh

　　所有节点检测，或者主节点宕掉，从节点提升为主节点，其他从节点修改主节点位置，都需要修改配置等，MHA Manager需要连接修改其配置，都需要ssh无密码登录，所以个节点能ssh免密码登录时第一步。


