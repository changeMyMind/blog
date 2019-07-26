# redis总结

---

快速生成数据的脚本 在课程中有的哦

#### 分布式锁

1. 什么是分布式锁？首先为了确保分布式锁可用，我们至少要确保锁的实现同时满足以下三个条件：
   1. 互斥性：在任意时刻，只有一个客户端能持有锁。
   2. 需要避免死锁：即使有一个客户端在持有锁的期间崩溃而没有主动解锁，也能保证后续其他客户端能加锁。
   3. 加锁解锁必须是同一个客户端：不能因为其他原因(如：业务时间超过设置的key的过期时间) 释放其他客户端的锁
2. 怎么实现分布式锁？ 实现分布式锁的方案有两种
   1. 采用lua脚本操作分布式锁
   2. 采用setnx、setex命令连用的方式实现分布式锁
3. 解锁需要注意什么
   1. 加锁和解锁必须是同一个客户端，如果客户端的锁在 程序未调用unlock(伪代码)的情况下锁失效了，在调用unlock的时候也不能释放其他客户端的锁

#### 读写分离

1. 海量并发性能瓶颈处理
2. 对读写进行扩展，采用读写分离方式解决性能瓶颈。运行一些额外的服务器，让他们跟主服务器进行连接，然后将请求分散到不同的服务器上面进行处理，用户可以从新添加的从服务器上获得额外的读查询处理能力
3. redis已经发现了这个读写分离场景特别普遍，自身集成了读写分离供用户使用。我们只需在redis的配置文件里面加上一条 `slaveof host port` 语句
4. 配置过程，启动多个redis节点，修改节点里面的redis.conf配置文件
5. 可能遇到的问题？
   1. 服务器下线导致数据丢失，slave下线之后怎么保证数据的同步

#### 主从构架数据同步

1. redis2.8之前的使用的是SYNC复制功能 非常消耗资源

   1. 主服务器需要执行BGSAVE命令来生成RDB文件，这个生成操作会消耗主服务器大量的CPU、内存和磁盘读写资源(bgsave是创建新线程 save命令会阻塞当前线程 有些配置文件的save 是表示 bgsave命令哦 如redis.conf中的save 表示的就是bgsave命令执行 lastsave命令可以展示上一次执行save的时间)
   2. 主服务器将RDB文件发送给从服务器，这个发送操作会耗费主从服务器大量的网络带宽和流量，并对主服务器响应命令请求的时间产生影响
   3. 接受到RDB文件的从服务器在载入文件的过程是阻塞的，无法处理命令请求

2. redis2.8后使用的是PSYNC赋值功能 特点：1. 完整重同步 full resynchronization 2. 部分同步 partial resynchronization

   1. 部分重同步功能步骤：

      1. 主服务器的复制偏移量(replication offset) 和从服务器的复制偏移量 比对 然后从服务器执行相应的命令

      2. 主服务器的复制积压缓冲区(replication backlog) 默认为1M

      3. 服务器的运行ID(run id)，用于存储服务器标识，

         如：从服务器断线重连，取到主服务器的运行ID与重连后主服务器运行ID进行对比，从而判断是执行部分重同步还是完整重同步 好处 从服务需要连接的是上一次的master节点 因为偏移量不同步可能会出现问题

   2. 第一次执行的时候就是完整重同步 或者 主服务器判断是否需要执行完整重同步还是执行部分重同步 判断依据：断线时间过长 导致不能从缓冲区获取所有执行命令的情况

#### reids 哨兵

- redis主节点挂掉之后应该怎么操作？命令模拟

```powershell
 slaveof no one       # 取消主备，变更为主节点

 slaveof 新host  新节点  # 将其他节点设置为新主节点的备份节点
```

- 开启Sentinel配置 3主3从 3主6从

```powershell
sentinel monitor mymaster 127.0.0.1 6379 1   
sentinel down-after-milliseconds mymaster 10000
sentinel failover-timeout mymaster 60000
sentinel parallel-syncs mymaster 1
```

- 命令讲解

  - sentinel monitor mymaster 127.0.0.1 6379 1 名称为mymaster的主节点名，1表示将这个主服务器判断为失效至少需要 1个 Sentinel 同意 （只要同意 Sentinel 的数量不达标，自动故障迁移就不会执行）
  - down-after-milliseconds 选项指定了 Sentinel 认为服务器已经断线所需的毫秒数
  - failover过期时间，当failover开始后，在此时间内仍然没有触发任何failover操作，当前sentinel 将会认为此次failoer失败
  - parallel-syncs 选项指定了在执行故障转移时， 最多可以有多少个从服务器同时对新的主服务器进行同步， 这个数字越小， 完成故障转移所需的时间就越长。
  - 如果从服务器被设置为允许使用过期数据集， 那么你可能不希望所有从服务器都在同一时间向新的主服务器发送同步请求， 因为尽管复制过程的绝大部分步骤都不会阻塞从服务器， 但从服务器在载入主服务器发来的 RDB 文件时， 仍然会造成从服务器在一段时间内不能处理命令请求： 如果全部从服务器一起对新的主服务器进行同步， 那么就可能会造成所有从服务器在短时间内全部不可用的情况出现。

- 启动所有主从上的sentinel

  - 前提是它们各自的server已成功启动 cd /usr/local/redis/src/redis-sentinel /etc/redis/sentinel.conf

- info Replication 查看节点信息

- shutdown主节点看服务是否正常

- Sentinel三大工作任务

  - 监控（Monitoring）： Sentinel 会不断地检查你的主服务器和从服务器是否运作正常。
  - 提醒（Notification）： 当被监控的某个 Redis 服务器出现问题时， Sentinel 可以通过 API 向管理员或者其他应用程序发送通知。
  - 自动故障迁移（Automatic failover）： 当一个主服务器不能正常工作时， Sentinel 会开始一次自动故障迁移操作， 它会将失效主服务器的其中一个从服务器升级为新的主服务器， 并让失效主服务器的其他从服务器改为复制新的主服务器； 当客户端试图连接失效的主服务器时， 集群也会向客户端返回新主服务器的地址， 使得集群可以使用新主服务器代替失效服务器。

- 互联网冷备和热备讲解，冷备和热备的特点分析

  - 冷备
    - 概念：冷备份发生在数据库已经正常关闭的情况下，当正常关闭时会提供给我们一个完整的数据库
    - 优点：是非常快速的备份方法（只需拷文件） 低度维护，高度安全
    - 缺点：单独使用时，只能提供到“某一时间点上”的恢复 而且再实施备份的全过程中，数据库必须要作备份而不能作其他工作。也就是说，在冷备份过程中，数据库必须是关闭状态
  - 热备
    - 概念：热备份是在数据库运行的情况下，采用archivelog mode方式备份数据库的方法
    - 优点：备份的时间短 备份时数据库仍可使用 可达到秒级恢复
    - 缺点：若热备份不成功，所得结果不可用于时间点的恢复 因难于维护，所以要非凡仔细小心

- **Sentinel是怎么工作的？**

  - 主观下线：

    - 概念主观下线（Subjectively Down， 简称 SDOWN）指的是单个 Sentinel 实例对服务器做出的下线判断

    - 主管下线特点：

      - 如果一个服务器没有在 master-down-after-milliseconds 选项所指定的时间内， 对向它发送 PING 命令的 Sentinel 返回一个有效回复（valid reply）， 那么 Sentinel 就会将这个服务器标记为主观下线

      - 服务器对 PING 命令的有效回复可以是以下三种回复的其中一种：

        ```shell
        返回 +PONG 。
        返回 -LOADING 错误。
        返回 -MASTERDOWN 错误。
        ```

  - 客观下线

    - 客观下线概念：
      - 指的是多个 Sentinel 实例在对同一个服务器做出 SDOWN 判断， 并且通过 SENTINEL is-master-down-by-addr 命令互相交流之后， 得出的服务器下线判断。 （一个 Sentinel 可以通过向另一个 Sentinel 发送 SENTINEL is-master-down-by-addr 命令来询问对方是否认为给定的服务器已下线。）
    - 客观下线特点：
      - 从主观下线状态切换到客观下线状态并没有使用严格的法定人数算法（strong quorum algorithm）， 而是使用了流言协议： 如果 Sentinel 在给定的时间范围内， 从其他 Sentinel 那里接收到了足够数量的主服务器下线报告， 那么 Sentinel 就会将主服务器的状态从主观下线改变为客观下线。 如果之后其他 Sentinel 不再报告主服务器已下线， 那么客观下线状态就会被移除。
    - 客观下线注意点：
      - 客观下线条件只适用于主服务器： 对于任何其他类型的 Redis 实例， Sentinel 在将它们判断为下线前不需要进行协商， 所以从服务器或者其他 Sentinel 永远不会达到客观下线条件。 只要一个 Sentinel 发现某个主服务器进入了客观下线状态， 这个 Sentinel 就可能会被其他 Sentinel 推选出， 并对失效的主服务器执行自动故障迁移操作。

#### 配置集群

```shell
# 集群部署需要redis-trib.rb 文件 因为是ruby写的所以需要安装ruby环境
yum -y install ruby  # -y 表示安装的时候 直接选y(yes)
# 下载到指定目录
cd /usr/local/
wget http://download.redis.io/releases/redis-4.0.14.tar.gz
# 解压
tar -zxvf redis-4.0.14.tar.gz
# 进入目录
cd redis-4.0.14
# 编译安装
make && make install
# 拷贝src下的redis-trib.rb 到 /usr/local/bin 目录下
cp /usr/local/redis-4.0.14/src/redis-trib.rb /usr/local/bin
# 在redis-4.0.14创建集群目录并进入
mkdir redis_cluster && cd redis_cluster
# 创建目录 6379 6380 6381 6382 6383 6384 因为需要至少6个节点哦
mkdir 6379 6380 6381 6382 6383 6384
# 复制配置文件
cp ../redis.conf 6379
cp ../redis.conf 6380
cp ../redis.conf 6381
cp ../redis.conf 6382
cp ../redis.conf 6383
cp ../redis.conf 6384
# 修改配置文件
vim 6379/redis.conf
  port 6379                            //端口      
  bind 127.0.0.1 172.17.172.23         //需要改为节点机器可访问的ip
  pidfile  ./redis_6379.pid            //pidfile文件对应相应的端口号
  daemonize yes                        //redis后台运行
  cluster-enabled  yes                 //开启集群 
  cluster-config-file nodes_6379.conf  //集群的配置，配置文件首次启动自动生成
  cluster-node-timeout 10000           //请求超时
  appendonly yes                       //aof日志开启志
vim 6380/redis.conf
  port 6380                            //端口      
  bind 127.0.0.1 172.17.172.23         //需要改为节点机器可访问的ip
  pidfile  ./redis_6380.pid            //pidfile文件对应相应的端口号
  daemonize yes                        //redis后台运行
  cluster-enabled  yes                 //开启集群 
  cluster-config-file nodes_6380.conf  //集群的配置，配置文件首次启动自动生成
  cluster-node-timeout 10000           //请求超时
  appendonly yes                       //aof日志开启志
vim 6381/redis.conf
  port 6381                            //端口      
  bind 127.0.0.1 172.17.172.23         //需要改为节点机器可访问的ip
  pidfile  ./redis_6381.pid            //pidfile文件对应相应的端口号
  daemonize yes                        //redis后台运行
  cluster-enabled  yes                 //开启集群 
  cluster-config-file nodes_6381.conf  //集群的配置，配置文件首次启动自动生成
  cluster-node-timeout 10000           //请求超时
  appendonly yes                       //aof日志开启志
vim 6382/redis.conf
  port 6382                            //端口      
  bind 127.0.0.1 172.17.172.23         //需要改为节点机器可访问的ip
  pidfile  ./redis_6382.pid            //pidfile文件对应相应的端口号
  daemonize yes                        //redis后台运行
  cluster-enabled  yes                 //开启集群 
  cluster-config-file nodes_6382.conf  //集群的配置，配置文件首次启动自动生成
  cluster-node-timeout 10000           //请求超时
  appendonly yes                       //aof日志开启志
vim 6383/redis.conf
  port 6383                            //端口      
  bind 127.0.0.1 172.17.172.23         //需要改为节点机器可访问的ip
  pidfile  ./redis_6383.pid            //pidfile文件对应相应的端口号
  daemonize yes                        //redis后台运行
  cluster-enabled  yes                 //开启集群 
  cluster-config-file nodes_6383.conf  //集群的配置，配置文件首次启动自动生成
  cluster-node-timeout 10000           //请求超时
  appendonly yes                       //aof日志开启志
vim 6384/redis.conf
  port 6384                            //端口      
  bind 127.0.0.1 172.17.172.23         //需要改为节点机器可访问的ip
  pidfile  ./redis_6384.pid            //pidfile文件对应相应的端口号
  daemonize yes                        //redis后台运行
  cluster-enabled  yes                 //开启集群 
  cluster-config-file nodes_6384.conf  //集群的配置，配置文件首次启动自动生成
  cluster-node-timeout 10000           //请求超时
  appendonly yes                       //aof日志开启志
# 启动
cd /usr/local/redis-4.0.14
./redis.server redis_cluster/6379/redis.conf
./redis.server redis_cluster/6380/redis.conf
./redis.server redis_cluster/6381/redis.conf
./redis.server redis_cluster/6382/redis.conf
./redis.server redis_cluster/6383/redis.conf
./redis.server redis_cluster/6384/redis.conf
# 集群启动 
redis-trib.rb create --replicas 1 39.97.109.173:6379 39.97.109.173:6380 39.97.109.173:6381 39.97.109.173:6382 39.97.109.173:6383 39.97.109.173:6384

# 关闭
redis-cli -p 6379 shutdown
redis-cli -p 6380 shutdown
redis-cli -p 6381 shutdown
redis-cli -p 6382 shutdown
redis-cli -p 6383 shutdown
redis-cli -p 6384 shutdown

# 添加节点 需要先添加对应的redis目录哦
# 添加主节点
redis-trib.rb add-node 127.0.0.1:6385 192.168.1.104:6386
# 添加从节点
redis-trib.rb add-node --slave 127.0.0.1:6387 127.0.0.1:6388
# 移动hash槽 跟着步骤会提示你输入 接受的节点Id是什么 node-id
redis-trib.rb reshard 127.0.0.1:6379 
# 移除节点 node-id 通过登录某一个节点然后输入 cluster nodes 第一列为node-id
redis-trib.rb del-node 127.0.0.1:6387 <node-id>
```



#### 集群搭建问题

除了ruby环境 搭建redis集群还需要 ruby-redis 接口

```shell
yum install gem 
gem install redis
# 可能会出现问题 因为yum目前支持的2.0.0的ruby下载(2019-05-31) 所以需要重新下载ruby哦
# 公钥私钥那一套 
gpg --keyserver hkp://keys.gnupg.net --recv-keys 409B6B1796C275462A1703113804BB82D39DC0E3 7D2BAF1CF37B13E2069D6956105BD0E739499BDB
# 下载安装
curl -sSL https://get.rvm.io | bash -s stable
# 查看文件是否齐全
find / -name rvm -print
# 一下显示的文件
				/usr/local/rvm
　　 　　/usr/local/rvm/src/rvm
　　 　　/usr/local/rvm/src/rvm/bin/rvm
　　 　　/usr/local/rvm/src/rvm/lib/rvm
　　 　　/usr/local/rvm/src/rvm/scripts/rvm
　　 　　/usr/local/rvm/bin/rvm
　　 　　/usr/local/rvm/lib/rvm
　　 　　/usr/local/rvm/scripts/rvm
# 使rvm生效
source /usr/local/rvm/scripts/rvm
# 查看ruby 版本
rvm list known
# 找一个版本下载 可能比较慢
rvm install 2.6.3
# 有可能需要卸载 使用 rvm remove 版本号(如：2.0.0) 
# 默认使用刚刚下载的ruby版本
rvm use 2.6.3 --default
# 再次执行下载接口命令
gem install redis
```



####  redis集群的数据分片

redis集群有16384个哈希槽(2^14),每个key通过CRC16校验后对16384取模来决定放置哪个槽，集群的每个节点负责一部分hash槽，举个例子，比如当前集群有3个节点，那么：(CRC16 是一种hash算法)

节点A约包含0到5500号hash槽

节点B约包含5501到11000号hash槽

节点C约包含11001到16384号hash槽

查看集群信息

```powershell
redis-cli -h 172.17.172.23 -p 6379 cluster nodes | grep master
```

这种结构很容易添加或者删除节点，比如如果我想新添加个节点D，我需要从节点A，B，C中得到部分槽到D上，如果我想移出A节点，需要将A节点上的所有hash槽移动到其他的节点上，然后将没有任何槽的A节点从集群环境中移出即可。由于从一个节点将hash槽移动到另一个节点并不会停止服务，所以无论添加删除或者改变某个节点的hash槽的数量都不会造成集群不可用的状态



#### redis的主从复制模型

为了防止大部分节点无法通信的情况下集群仍然可用，所以集群使用了主从复制模型，每个节点都会有N-1个复制品



#### redis一致性保证

主节点对命令的复制工作发生在返回命令回复之后，因为如果每次处理命令请求都需要等待复制操作完成的话，那么主节点处理命令请求的速度将极大地降低，我们必须在性能和一致性之间做出权衡。注意：redis集群可能会在将来提供同步写的方法。redis集群另一种可能会丢失命令的情况是集群出现了网络分区，并且一个客户端与至少包括一个主节点在内的少数实例被孤立

#### redis测试故障转移

```powershell
redis-cli -h 172.17.172.23 -p 6379 debug segfault
redis-cli -h 172.17.172.23 -p 6379 cluster nodes | grep master
```

#### redis cluster 集群方式原理

1. 数据拆分问题

   水平切分于垂直切分相比，相对来说稍微复杂一些。因为要将同一个表中的不同数据拆分到不同的数据库中。

   分片是一种基于数据库分成若干片段的传统概念扩容技术，它将数据库分割成多个碎片并将这些碎片放置在不同的服务器上。

    垂直拆分最大的特点就是规则简单，实施方便，尤其适合各业务之间耦合度非常低，影响比较小，业务逻辑非常清晰的系统，按照不同的业务纬度将不同数据放入不同的表

2. 故障转移

   通过gossip节点信息同步实现，实现slave作为master的被分界点，并实现故障剔除master节点采用slave替换，饼子啊替换完之后将结果通知到其他节点上

3. Twitter推特公司twemproxy服务端分片和客户端分片集群解决方案

#### redis的持久化方式

两种 RDB、AOF 前者快照保存所有 后者指令命令



#### redis 过期key清除策略

1. 惰性删除

   Key被访问的时候会判断这个key是否过期 如果过期删除key

   特点：CPU友好，但是浪费内存资源，并且如果一个key不再使用，那么它会一直存在内存中，造成资源浪费

2. 定时删除

   设置建的过期时间的同时，创建一个定时器(timer)，让定时器在键的过期时间来临时，立即执行对键的删除操作

3. 定期删除

    隔一段时间，程序就对数据库进行一次检查，删除里面的过期键，至于要删除多少过期键，以及要检查多少个数据库，又算法决定。即设置一个定时任务，比如10分钟删除一次过期的key；间隔小则占用cpu，间隔大则浪费内存

      例如redis每秒处理：

   	1. 测试随机的20个keys进行相关过期检测。
    	2. 删除所有已经过期的keys
    	3. 如果有多于25%的keys过期，重复步骤1

总结：redis服务器实际使用的是惰性删除和定期删除两种策略：通过配合使用这两种删除策略，服务器可以很好的在合理使用CUP时间和避免内存资源浪费之间取得平衡

惰性删除策略怎么实现：expirelfNeeded函数，当我们操作key的时候进行判断key是否过期

定期删除策略怎么实现：用过activeExpireCycle函数，serverCron函数（定时任务）执行时，activeExpireCycle函数就会被调用，规定的时间里面分多次遍历服务器的expires字典随机检查一部分key的过期时间，并删除其中的过期key