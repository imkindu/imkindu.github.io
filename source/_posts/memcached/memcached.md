---
title: memcached简介
date: 2017-11-06 09:00:00
author: imkindu
categories: memcached
tags:
- memcached
---


高效缓存

<!--more-->



## memcached

[root@centos55 ~]# rpm -ql memcached
/etc/rc.d/init.d/memcached
/etc/sysconfig/memcached
/usr/bin/memcached
/usr/bin/memcached-tool
/usr/share/doc/memcached-1.4.4
/usr/share/doc/memcached-1.4.4/AUTHORS
/usr/share/doc/memcached-1.4.4/CONTRIBUTORS
/usr/share/doc/memcached-1.4.4/COPYING
/usr/share/doc/memcached-1.4.4/ChangeLog
/usr/share/doc/memcached-1.4.4/NEWS
/usr/share/doc/memcached-1.4.4/README
/usr/share/doc/memcached-1.4.4/protocol.txt
/usr/share/doc/memcached-1.4.4/readme.txt
/usr/share/doc/memcached-1.4.4/threads.txt
/usr/share/man/man1/memcached.1.gz
/var/run/memcached




[root@centos55 ~]# memcached -help
memcached 1.4.4
-p <num>      TCP port number to listen on (default: 11211)
-U <num>      UDP port number to listen on (default: 11211, 0 is off)
-s <file>     UNIX socket path to listen on (disables network support)
-a <mask>     access mask for UNIX socket, in octal (default: 0700)
-l <ip_addr>  interface to listen on (default: INADDR_ANY, all addresses)
-d            run as a daemon
-r            maximize core file limit
-u <username> assume identity of <username> (only when run as root)
-m <num>      max memory to use for items in megabytes (default: 64 MB)
-M            return error on memory exhausted (rather than removing items)
-c <num>      max simultaneous connections (default: 1024)
-k            lock down all paged memory.  Note that there is a
              limit on how much memory you may lock.  Trying to
              allocate more than that would fail, so be sure you
              set the limit correctly for the user you started
              the daemon with (not for -u <username> user;
              under sh this is done with 'ulimit -S -l NUM_KB').
-v            verbose (print errors/warnings while in event loop)
-vv           very verbose (also print client commands/reponses)
-vvv          extremely verbose (also print internal state transitions)
-h            print this help and exit
-i            print memcached and libevent license          # 显示许可证
-P <file>     save PID in <file>, only used with -d option
-f <factor>   chunk size growth factor (default: 1.25)
-n <bytes>    minimum space allocated for key+value+flags (default: 48)
-L            Try to use large memory pages (if available). Increasing
              the memory page size could reduce the number of TLB misses
              and improve the performance. In order to get large pages
              from the OS, memcached will allocate the total item-cache
              in one large chunk.
-D <char>     Use <char> as the delimiter between key prefixes and IDs.
              This is used for per-prefix stats reporting. The default is
              ":" (colon). If this option is specified, stats collection
              is turned on automatically; if not, then it may be turned on
              by sending the "stats detail on" command to the server.
-t <num>      number of threads to use (default: 4)         # 线程数
-R            Maximum number of requests per event, limits the number of
              requests process for a given connection to prevent 
              starvation (default: 20)
-C            Disable use of CAS
-b            Set the backlog queue limit (default: 1024)
-B            Binding protocol - one of ascii, binary, or auto (default) # 二进制格式
-I            Override the size of each slab page. Adjusts max item size
              (default: 1mb, min: 1k, max: 128m)


## memcached-tool


memcached-tool <host[:port] | /path/to/socket> [mode]

Usage: memcached-tool <host[:port]> [mode]

       memcached-tool 10.0.0.5:11211 display    # shows slabs
       memcached-tool 10.0.0.5:11211            # same.  (default is display)
       memcached-tool 10.0.0.5:11211 stats      # shows general stats
       memcached-tool 10.0.0.5:11211 dump       # dumps keys and values


MODES
       display
              Print slab class statistics. This is the default mode if no mode is specified.  The printed columns are:

              #      Number of the slab class.
              Item_Size
                     The amount of space each chunk uses. One item uses one chunk of the appropriate size.
              Max_age
                     Age of the oldest item in the LRU.
              Pages  Total number of pages allocated to the slab class.
              Count  Number of items presently stored in this class. Expired items are not automatically excluded.
              Full?  Yes if there are no free chunks at the end of the last allocated page.
              Evicted
                     Number of times an item had to be evicted from the LRU before it expired.
              Evict_Time
                     Seconds since the last access for the most recent item evicted from this class.
              OOM    Number of times the underlying slab class was unable to store a new item.
       stats  Print general-purpose statistics of the daemon. Each line contains the name of the statistic and its value.
       dump   Make a partial dump of the cache written in the add statements of the memcached protocol.




/usr/share/doc/memcached-1.4.4/protocol.txt 查看命令



memcached session manager