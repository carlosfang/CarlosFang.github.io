---
layout: post
title: redis 配置说明
date: 2015-02-04 15:08:12
categories: redis
tags: db redis
excerpt: redis configuration
---

### Redis 支持写的指令
Redis大概的命令如下：

>set setnx setex append
incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
getset mset msetnx exec sort

### Redis 配置
 
1. redis 配置文件一般是在/etc/redis.conf
2. redis官方提供的redis.conf文件，足有700+行，其中100多行为有效配置行，另外的600多行为注释说明。
3. redis配置中对单位的大小写不敏感，1GB、1Gb和1gB都是相同的。由此也说明，redis只支持bytes，不支持bit单位。
4. redis支持“主配置文件中引入外部配置文件  include /path/to/other.conf
5. redis 开头的单位转换：（从配置文件中copy出来）
 ```sh
    # 1k => 1000 bytes
    # 1kb => 1024 bytes
    # 1m => 1000000 bytes
    # 1mb => 1024*1024 bytes
    # 1g => 1000000000 bytes
    # 1gb => 1024*1024*1024 bytes
    #
    # units are case insensitive so 1GB 1Gb 1gB are all the same.
 ```
6. redis配置文件被分成了几大块区域:
 ```sh
    1)通用（general）
    2)快照（snapshotting）
    3)复制（replication）
    4)安全（security）
    5)限制（limits)
    6)追加模式（append only mode)
    7)LUA脚本（lua scripting)
    8)慢日志（slow log)
    9)事件通知（event notification）
 ```
### 通用

1 daemon  ： redis 是可以以后台服务运行的。

 以daemon形式运行时，redis会生成一个pid文件，默认会生成在/var/run/redis.pid。当然，你可以通过pidfile来指定pid文件生成的位置。

2 pidfile ：配置pid文件路径 pidfile /path/to/redis.pid

3 bind    ：redis会响应本机所有可用网卡的连接请求。当然，redis允许你通过bind配置项来指定要绑定的IP  bind 192.168.1.2 10.8.4.2

4 port    ：redis的默认服务端口是6379，你可以通过port配置项来修改。如果端口设置为0的话，redis便不会监听端口了。

5 unixsocket  ：如果redis不监听端口，redis还支持通过unix socket方式来接收请求。 可以通过unixsocket配置项来指定unix socket文件的路径，并通过unixsocketperm来指定文件的权限

6 unixsocketperm : 来指定文件的权限;

7 timeout ： 当一个redis-client一直没有请求发向server端，那么server端有权主动关闭这个连接，可以通过timeout来设置“空闲超时时限”，0表示永不关闭。

8 tcp-keepalive  ：TCP连接保活策略，可以通过tcp-keepalive配置项来进行设置，单位为秒。  假如设置为60秒，则server端会每60秒向连接空闲的客户端发起一次ACK请求，以检查客户端是否已经挂掉，对于无响应的客户端则会关闭其连接。  所以关闭一个连接最长需要120秒的时间。如果设置为0，则不会进行保活检测。

9  loglevel ：配置项设置日志等级，共分四级，即debug、verbose、notice、warning。

10 logfile  ：设置日志文件的生成位置。如果设置为空字符串，则redis会将日志输出到标准输出。
 假如你在daemon情况下将日志设置为输出到标准输出，则日志会被写到/dev/null中。

11 databases  ：设置其数据库的总数量，假如你希望一个redis包含16个数据库。 
 数据库的编号将是0到15。默认的数据库是编号为0的数据库。用户可以使用select <DBid>来选择相应的数据库。

12 syslog-ident   ： 不了解

13 syslog-facility： 不了解

### 快照 （主要涉及的是redis的RDB持久化相关的配置）

1 save <seconds> <changes>  ：让数据保存到磁盘上，即控制RDB快照功能。
如果你想禁用RDB持久化的策略，只要不设置任何save指令就可以，或者给save传入一个空字符串参数也可以达到相同效果　

如：
```sh
 // --------------------------------------------------------
 #save 900 1     // 表示每15分钟且至少有1个key改变，就触发一次持久化
 #save 300 10    // 表示每5分钟且至少有10个key改变，就触发一次持久化
 #save 60  10000 // 表示每60秒至少有10000个key改变，就触发一次持久化
 #save ""        // 禁用
// --------------------------------------------------------
```

2 stop-writes-on-bgsave-error ： 如果开启了RDB快照功能，那么在redis持久化数据到磁盘时如果出现失败，默认情况下，redis会停止接受所有的写请求 

3 rdbcompression ：是否进行压缩。

4 rdbchecksum  ：是否CRC检验， 如果要有最大性能， 可以关闭，大概消10%的性能。
 
5 dbfilename  ：快照文件的名称，默认是这样配置的。
6 dir   ： 快照文件存放的路径。比如默认设置就是当前文件夹

### 复制 （就是主从同步功能）

 1 slaveof <masterip> <masterport> :  通过slaveof配置项可以控制某一个redis作为另一个redis的从服务器，通过指定IP和端口来定位到主redis的位置。
    
 一般情况下，我们会建议用户为从redis设置一个不同频率的快照持久化的周期，或者为从redis配置一个不同的服务端口等等。
       
2 masterauth <master-password>    : 如果主redis设置了验证密码的话（使用requirepass来设置），则在从redis的配置中要使用masterauth来设置校验密码，否则的话，主redis会拒绝从redis的访问请求。
        
  *当从redis失去了与主redis的连接，或者主从同步正在进行中时，redis该如何处理外部发来的访问请求呢？
  
  这里，从redis可以有两种选择：
   a)如果slave-serve-stale-data设置为yes（默认），则从redis仍会继续响应客户端的读写请求。       
　 b)如果slave-serve-stale-data设置为no，则从redis会对客户端的请求返回“SYNC with master in progress”， 
当然也有例外，当客户端发来INFO请求和SLAVEOF请求，从redis还是会进行处理。

３ 你可以控制一个从redis是否可以接受写请求。将数据直接写入从redis，一般只适用于那些生命周期非常短的数据，因为在主从同步时，这些临时数据就会被清理掉。

 自从redis2.6版本之后，默认从redis为只读。

 ```sh
  # slave-read-only yes 
 ```
                     
### 安全 

1 requirepass ： 设置密码。 由于redis性能非常高，所以每秒钟可以完成多达15万次的密码尝试，所以你最好设置一个足够复杂的密码，否则很容易被黑客破解
2  rename-command  ： redis允许我们对redis指令进行更名，比如将一些比较危险的命令改个名字，避免被误执行。
 比如可以把CONFIG命令改成一个很复杂的名字，这样可以避免外部的调用，同时还可以满足内部调用的需要：

 ```sh
rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c89 //  命令重命名
rename-command CONFIG ""                                       //  禁用
```

###  限制

1  maxclients ： 设置redis同时可以与多少个客户端进行连接。默认情况下为10000个客户端。
    当你无法设置进程文件句柄限制时，redis会设置为当前的文件句柄限制值减去32，因为redis会为自身内部处理逻辑留一些句柄出来。

2 maxmemory ： 设置redis可以使用的内存量。 

一旦到达内存使用上限，redis将会试图移除内部数据，移除规则可以通过maxmemory-policy来指定。

需要注意的一点是，如果你的redis是主redis（说明你的redis有从redis），那么在设置内存使用上限时，

需要在系统中留出一些内存空间给同步队列缓存，只有在你设置的是“不移除”的情况下，才不用考虑这个因素。

3 maxmemory-policy： 内存移除策略。

 对于内存移除规则来说，redis提供了多达6种的移除规则。他们是：

  1) volatile-lru：使用LRU算法移除过期集合中的key
  
  2) allkeys-lru：使用LRU算法移除key
   
  3) volatile-random：在过期集合中移除随机的key

  4) allkeys-random：移除随机的key

  5) volatile-ttl：移除那些TTL值最小的key，即那些最近才过期的key。
 
  6) noeviction：不进行移除。针对写操作，只是返回错误信息。

###  追加模式

1 默认情况下，redis会异步的将数据持久化到磁盘。这种模式在大部分应用程序中已被验证是很有效的。
   但是在一些问题发生时，比如断电，则这种机制可能会导致数分钟的写请求丢失。追加文件（Append Only File）是一种更好的保持数据一致性的方式。

 即使当服务器断电时，也仅会有1秒钟的写请求丢失，当redis进程出现问题且操作系统运行正常时，甚至只会丢失一条写请求。
                             　
2  appendonly ：开关   // appendonly　no  关闭追加功能

3 appendfilename　：文件名 // appendfilename  "appendonly.aof"
       
4 appendfsync  : fsync()调用方式 默认情况下为everysec  // appendfsync everysec 

 redis支持三种不同的模式：

  a) no：不调用fsync()。而是让操作系统自行决定sync的时间。这种模式下，redis的性能会最快。
  b) always：在每次写请求后都调用fsync()。这种模式下，redis会相对较慢，但数据最安全。
  c) everysec：每秒钟调用一次fsync()。这是性能和安全的折衷。
        
5 当fsync方式设置为always或everysec时，如果后台持久化进程需要执行一个很大的磁盘IO操作，那么redis可能会在fsync()调用时卡住。
  目前尚未修复这个问题，这是因为即使我们在另一个新的线程中去执行fsync()，也会阻塞住同步写调用。

6 为了缓解这个问题，我们可以使用下面的配置项，这样的话，当BGSAVE或BGWRITEAOF运行时，fsync()在主进程中的调用会被阻止。
    这意味着当另一路进程正在对AOF文件进行重构时，redis的持久化功能就失效了，就好像我们设置了“appendsync none”一样。如果你的redis有时延问题，那么请将下面的选项设置为yes。否则请保持no，因为这是保证数据完整性的最安全的选择. 
```sh  
no-appendfsync-on-rewrite no   
```
7 只读的从redis并不适合直接暴露给不可信的客户端。为了尽量降低风险，可以使用rename-command指令来将一些可能有破坏力的命令重命名，避免外部直接调用。比如：
```sh
 rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52
```        
    
8 repl-ping-slave-period : 从redis会周期性的向主redis发出PING包。你可以通过repl_ping_slave_period指令来控制其周期。默认是10秒。
```sh 
 # repl-ping-slave-period 10
``` 
    
9  repl-timeout  :  主从之间超时的时限， #repl-timeout 60
   
   在主从同步时，可能在这些情况下会有超时发生：

   a)以从redis的角度来看，当有大规模IO传输时。

   b)以从redis的角度来看，当数据传输或PING时，主redis超时.

   c)以主redis的角度来看，在回复从redis的PING时，从redis超时.

10 repl-disable-tcp-nodelay :  控制在主从同步时是否禁用TCP_NODELAY。如果开启TCP_NODELAY，那么主redis会使用更少的TCP包和更少的带宽来向从redis传输数据。但是这可能会增加一些同步的延迟，大概会达到40毫秒左右。
 如果你关闭了TCP_NODELAY，那么数据同步的延迟时间会降低，但是会消耗更多的带宽。

 ```sh
    # repl-disable-tcp-nodelay no
```

11  repl-backlog-size :  设置同步队列长度。队列长度（backlog)是主redis中的一个缓冲区，在与从redis断开连接期间，
  主redis会用这个缓冲区来缓存应该发给从redis的数据。这样的话，当从redis重新连接上之后，就不必重新全量同步数据，只需要同步这部分增量数据即可. 
```sh
# repl-backlog-size 1mb
```

12 repl-backlog-ttl :  如果主redis等了一段时间之后，还是无法连接到从redis，那么缓冲队列中的数据将被清理掉。 我们可以设置主redis要等待的时间长度。如果设置为0，则表示永远不清理。默认是1个小时。  

```sh
  # repl-backlog-ttl 3600  
```                      
                                   
13 slave-priority  : 我们可以给众多的从redis设置优先级，在主redis持续工作不正常的情况，优先级高的从redis将会升级为主redis。
     而编号越小，优先级越高。比如一个主redis有三个从redis，优先级编号分别为10、100、25，那么编号为10的从redis将会被首先选中升级为主redis。 当优先级被设置为0时，这个从redis将永远也不会被选中。默认的优先级为100。  

```sh 
 # slave-priority 100    
```                    
                                   
14  假如主redis发现有超过M个从redis的连接延时大于N秒，那么主redis就停止接受外来的写请求。
 这是因为从redis一般会每秒钟都向主redis发出PING，而主redis会记录每一个从redis最近一次发来PING的时间点，
 所以主redis能够了解每一个从redis的运行情况。 

 ```sh                         
# min-slaves-to-write 3
# min-slaves-max-lag 10

# 上面这个例子表示，假如有大于等于3个从redis的连接延迟大于10秒，那么主redis就不再接受外部的写请求。
# 上述两个配置中有一个被置为0，则这个特性将被关闭。默认情况下min-slaves-to-write为0，而min-slaves-max-lag为10。
```
                                   
    