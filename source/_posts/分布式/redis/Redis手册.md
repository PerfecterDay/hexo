---
title: Redis 手册
date: 2019-05-25 21:07:35
tags: 分布式
category: 分布式
typora-root-url: ..\..\..
---

Redis是一种基于键值对（key-value）的NoSQL数据库，与很多键值对数据库不同的是，Redis中的值可以是由string（字符串）、hash（哈希）、list（列表）、set（集合）、zset（有序集合）、Bitmaps（位图）、HyperLogLog、GEO（地理信息定位）等多种数据结构和算法组成，因此Redis可以满足很多的应用场景，而且因为Redis会将所有数据都存放在内存中，所以它的读写性能非常惊人。不仅如此，Redis还可以将内存的数据利用快照和日志的形式保存到硬盘上，这样在发生类似断电或者机器故障的时候，内存中的数据不会“丢失”。除了上述功能以外，Redis还提供了键过期、发布订阅、事务、流水线、Lua脚本等附加功能。总之，如果在合适的场景使用好Redis，它就会像一把瑞士军刀一样所向披靡。

### Redis Server 启动与连接
<img src="/pics/redis-exe.png" alt="">

1. redis-server
   1. 默认配置启动：`# redis-server`
   2. 指定配置文件启动: `# redis-server /opt/redis/redis.conf`
3. 运行时指定配置启动：`# redis-server --configKey1 configValue1 --configKey2 configValue2`
   

Redis目录下都会有一个redis.conf配置文件，里面就是Redis的默认配置，通常来讲我们会在一台机器上启动多个Redis，并且将配置集中管理在指定目录下，而且配置不是完全手写的，而是将redis.conf作为模板进行修改。

   一些基础配置：
   + port : 配置监听端口
   + logfile : 配置日志文件
   + dir : Redis 工作目录（存放持久化文件和日志文件）
+ daemonize: 是否以守护进程的方式启动
  
2. redis-cli
   + `redis-cli -h {host} -p {port} -a {password}` ：使用主机、端口和密码以交互式连接 redis 服务
   + `redis-cli -h {host} -p {port} -a {password} {command}` ：使用主机、端口和密码以命令式连接 redis 服务并执行一条命令

3. 关闭 Redis 服务
   1. `redis-cli shutdown`：断开与客户端的连接、持久化文件生成，是一种相对优雅的关闭方式。
   2. shutdown还有一个参数，代表是否在关闭Redis前，生成持久化文件：`redis-cli shutdown nosave|save`
   3. 除了可以通过shutdown命令关闭Redis服务以外，还可以通过kill进程号的方式关闭掉Redis，但是不要粗暴地使用kill-9强制杀死Redis服务，不但不会做持久化操作，还会造成缓冲区等资源不能被优雅关闭，极端情况会造成AOF和复制丢失数据的情况。

### 常用命令

1. 配置命令
   1. 查看所有配置： `config get *`
   2. 查看指定配置：`config get XXX`
   3. 设置config： `config set xxx xxx`
2. 键通用命令
   1. 查看所有的键： `keys *`，会遍历所有的键，谨慎使用。
   2. 键总数： `dbsize` ,不会遍历所有键，而是直接获取 Redis内置的键总数变量的值
   3. 检查键是否存在： `exists key`， 存在返回1，否则返回0
   4. 删除键： `del key1 key2 ...`, 返回成功删除键的个数
   5. 键过期： `expire key seconds` ,超过过期时间后，键会自动删除
   6. 键剩余过期时间： `ttl key`，返回大于等于0的整数，代表剩余过期时间；如果没有设置过期时间，返回-1；键不存在，返回-2
   7. 查看键对应的值的数据类型： `type key`，键不存在返回 none
   8. 键重命名： `rename key newkey`, 如果 newkey 已经存在，那么他的值将会被 key 的值覆盖
   9. `renamenx key newkey`： 只有 newkey 不存在时才会重命名成功，由于重命名键期间会执行del命令删除旧的键，如果键对应的值比较大，会存在阻塞Redis的可能性，这点不要忽视。
   10. 随机返回一个键： `randomkey`


### 数据类型

#### 字符串
字符串类型是 Redis 最基础的类型，所有的键都是字符串类型。字符串的值实际可以是字符串、数字，甚至二进制（图片、音视频），但是大小最大不能超过512MB。
<img src="/pics/redis-string.png" alt="">

1. 命令
   + `set key value [ex seconds] [px milliseconds] [nx|xx]` : 设置值
   + `setex` : 相当于上面的 ex 选项
   + `setnx` ：相当于上面的 nx 选项，不存在时才设置成功。
   + `mset key value [key value...]` : 批量设置值
   + `get key` ：获取 key 对应的值，key 不存在则返回 nil。
   + `mget key [key...]` : 批量获取值
   + `incr key` ：计数自增,若值不是整数，返回错误；若值是整数，返回自增后的结果；键不存在，按照值为0自增，返回结果1。
   + `decr key` ：计数递减1
   + `incrby key increment` ：计数 + increment
   + `decrby key decrement` ：计数 - decrement
   + `incrbyfloat key increment` ：浮点数 + increment
   + `append key value` ：向字符串尾部追加值
   + `strlen value` ：获取字符串长度
   + `getset key value` ：设置并返回原值
   + `setrange key offeset value` ：设置指定位置的字符
   + `getrange key start end` ：获取部分字符串
   <img src="/pics/redis-string-time.png" alt="">

2. 内部编码
   字符串类型的内部编码有3种：
   + int：8个字节的长整型。
   + embstr：小于等于39个字节的字符串。
   + raw：大于39个字节的字符串。

#### 哈希
<img src="/pics/redis-hash.png" alt="">

1. 命令
   + 设置值 ：`hset key field value`,设置 field-value，成功返回1，否则返回0。还有 `hsetnx` 命令，只有field不存在时才会设置值
   + 获取值 ：`hget key field`；如果 field 不存在，返回 nil
   + 删除 field ：`hdel key field [field ...]`：可以删除一个或多个 field，返回成功删除的个数
   + 计算field的个数 ：`hlen key`
   + 批量设置field-value ：`hmset key field value [field value ...]`
   + 批量获取field-value ：`hmget key field [field....]`
   + 判断 field 是否存在 ：`hexists key field`,存在返回1，否则返回0
   + 获取所有 field ：`hkeys key`
   + 获取所有 value ：`hvals key`
   + 获取所有 field-value ：`hgetall key`
   + 指定 field 加1：`hincrby key field`
   + 同上，对浮点数操作 ：`hincrbyfloat key field`
   + 计算value的字符串长度 ：`hstrlen key field`

   <img src="/pics/redis-hash-time.png" alt="">


#### 列表
#### 集合
#### 有序集合

### Jedis 简介
1. 直接构造操作 Jedis 
   ```
   # 1. 生成一个Jedis对象，这个对象负责和指定Redis实例进行通信
   Jedis jedis = new Jedis("127.0.0.1", 6379);
   # 2. jedis执行set操作
   jedis.set("hello", "world");
   # 3. jedis执行get操作, value="world"
   String value = jedis.get("hello");
   ```
   还有一个包含4个参数的构造函数是比较常用的：
   `Jedis(final String host, final int port, final int connectionTimeout, final int
   soTimeout)`
   + host：Redis实例的所在机器的IP。
   + port：Redis实例的端口。
   + connectionTimeout：客户端连接超时。
   + soTimeout：客户端读写超时。

   Jedis 对 Redis 五种数据结构的操作示例：
   ```
   // 1.string
   // 输出结果：OK
   jedis.set("hello", "world");
   // 输出结果：world
   jedis.get("hello");
   // 输出结果：1
   jedis.incr("counter");
   // 2.hash
   jedis.hset("myhash", "f1", "v1");
   jedis.hset("myhash", "f2", "v2");
   // 输出结果：{f1=v1, f2=v2}
   jedis.hgetAll("myhash");
   // 3.list
   jedis.rpush("mylist", "1");
   jedis.rpush("mylist", "2");
   jedis.rpush("mylist", "3");
   // 输出结果：[1, 2, 3]
   jedis.lrange("mylist", 0, -1);
   // 4.set
   jedis.sadd("myset", "a");
   jedis.sadd("myset", "b");
   jedis.sadd("myset", "a");
   // 输出结果：[b, a]
   jedis.smembers("myset");
   // 5.zset
   jedis.zadd("myzset", 99, "tom");
   jedis.zadd("myzset", 66, "peter");
   jedis.zadd("myzset", 33, "james");
   // 输出结果：[[["james"],33.0], [["peter"],66.0], [["tom"],99.0]]
   jedis.zrangeWithScores("myzset", 0, -1);
   ```
2. Jedis 连接池
   Jedis提供了JedisPool这个类作为对Jedis的连接池，同时使用了Apache的通用对象池工具common-pool作为资源的管理工具，下面是使用JedisPool操作Redis的代码示例：
   ```
   // common-pool连接池配置，这里使用默认配置，后面小节会介绍具体配置说明
   GenericObjectPoolConfig poolConfig = new GenericObjectPoolConfig();
   // 初始化Jedis连接池
   JedisPool jedisPool = new JedisPool(poolConfig, "127.0.0.1", 6379);

   //从连接池获取 Jedis 连接对象
   Jedis jedis = null;
   try {
      // 1. 从连接池获取jedis对象
      jedis = jedisPool.getResource();
      // 2. 执行操作
      jedis.get("hello");
   } catch (Exception e) {
      logger.error(e.getMessage(),e);
   } finally {
      if (jedis != null) {
         // 如果使用JedisPool，close操作不是关闭连接，代表归还连接池
         jedis.close();
      }
   }
   ```

   下边是 Jedis 连接池的配置参数：
   <img src="/pics/jedispool-config.png" alt="">

### Redis 持久化策略
Redis 支持两种持久化方式：
+  RDB,Snapshoting (快照，默认方式)
+  Append-only file (AOF)

#### RDB
RDB是默认的持久化方式。这种方式是就是将内存中数据以快照的方式写入到二进制文件中,默认的文件名为 `dump.rdb` 。

##### RDB 触发方式

触发RDB持久化过程分为手动触发和自动触发。

1. 手动触发
   手动触发分别对应 save 和 bgsave 命令：
   + save 命令：对于单线程的 Redis 服务器，会阻塞服务器，直到 RDB 过程完成为止，对于内存中存放大量数据的实例会造成长时间阻塞，线上环境不建议使用。
   + bgsave 命令：Redis 进程执行 fork 操作创建子进程，RDB过程由子进程负责，完成后自动结束，阻塞只发生在 fork 阶段，一般时间很短。
2. 自动触发
   下述场景会自动触发 RDB 持久化机制：
   1. 通过配置设置了自动做快照持久化的方式。我们可以配置 redis 在 n 秒内如果超过 m 个 key 被修改就自动做快照（bgsave），下面是默认的快照保存配置：
      + save 900 1 #900 秒内如果超过 1 个 key 被修改，则发起快照保存
      + save 300 10 #300 秒内容如超过 10 个 key 被修改，则发起快照保存
      + save 60 10000 #60 秒内容如超过 10000 个 key 被修改，则发起快照保存
   2. 如果从节点执行全量复制操作，主节点会自动执行 bgsave 生成 RDB 文件发送给从节点
   3. 执行 debug reload 命令重新加载 Redis 时，也会自动触发
   4. 默认情况下执行 shutdown 命令时，如果没有开启 AOF 持久化功能则自动执行 bgsave 。

##### RDB快照保存过程

1. redis 调用 fork后,于是有了子进程和父进程。
2. 父进程继续处理 client 请求，子进程负责将内存内容写入到临时文件。由于 os 的实时复制机制（ copy on write)父子进程会共享相同的物理页面，当父进程处理写请求时 os 会为父进程要修改的页面创建副本，而不是写共享的页面。所以子进程地址空间内的数据是 fork 时刻整个数据库的一个快照。
3. 当子进程将快照写入临时文件完毕后，用临时文件替换原来的快照文件，然后子进程退出。 client 也可以使用 `save` 或者 `bgsave` 命令通知 redis 做一次快照持久化。 save 操作是在主线程中保存快照的，由于 redis 是用一个主线程来处理所有 client 的请求，这种方式会阻塞所有client 请求。所以不推荐使用。另一点需要注意的是，每次快照持久化都是将内存数据完整写入到磁盘一次，并不是增量的只同步变更数据。如果数据量大的话，而且写操作比较多，必然会引起大量的磁盘 io 操作，可能会严重影响性能。

##### RDB文件的处理

RDB文件保存在`dir`配置指定的目录下，文件名通过 `dbfilename` 配置指定。可以通过执行 `config set dir{newDir}` 和 `config set dbfilename {newFileName}` 运行期动态执行，当下次运行时RDB文件会保存到新目录。

Redis默认采用LZF算法对生成的RDB文件做压缩处理，压缩后的文件远远小于内存大小，默认开启，可以通过参数 `config set rdbcompression {yes|no}` 动态修改。

如果Redis加载损坏的RDB文件时拒绝启动，可以使用Redis提供的 `redis-check-dump` 工具检测RDB文件并获取对应的错误报告。

##### RDB的优缺点

1. RDB的优点：
   + RDB是一个紧凑压缩的二进制文件，代表Redis在某个时间点上的数据快照。非常适用于备份，全量复制等场景。比如每6小时执行bgsave备份，并把RDB文件拷贝到远程机器或者文件系统中（如hdfs），用于灾难恢复。
   + Redis加载RDB恢复数据远远快于AOF的方式。
2. RDB的缺点：
   + RDB方式数据没办法做到实时持久化/秒级持久化。因为bgsave每次运行都要执行fork操作创建子进程，属于重量级操作，频繁执行成本过高。
   + RDB文件使用特定二进制格式保存，Redis版本演进过程中有多个格式的RDB版本，存在老版本Redis服务无法兼容新版RDB格式的问题。


#### AOF
首先通过 `appendonly yes` 启用 aof 持久化方式。AOF 以独立日志的方式记录每次写命令。重启时再重新执行AOF文件中的命令达到恢复数据的目的。AOF的主要作用是解决了数据持久化的实时性，目前已经是Redis持久化的主流方式。
由于快照方式是在一定间隔时间做一次的，所以如果 redis 意外 down 掉的话，就会丢失最后一次快照后的所有修改。如果应用要求不能丢失任何修改的话，可以采用 aof 持久化方式。

##### AOF  刷盘配置

aof 比快照方式有更好的持久化性能，是由于在使用 aof 持久化方式时,redis 会将每一个收到的写命令都通过 write 函数追加到日志文件中(默认是 appendonly.aof)。当 redis 重启时会通过重新执行文件中保存的写命令来在内存中重建整个数据库的内容。当然由于 os 会在内核中缓存（AOF缓冲区） write 做的修改，所以可能不是立即写到磁盘上。这样 aof 方式的持久化也还是有可能会丢失部分修改。不过我们可以通过配置文件告诉 redis 我们想要通过 fsync 函数强制 os 写入到磁盘的时机。有三种方式如下（默认是：每秒 fsync 一次）：

1. appendfsync always //收到写命令写入aof_buf后调用 fsync 就立即写入磁盘，fsync 完成后线程返回。最慢，但是保证完全的持久化
2. appendfsync everysec //默认方式，，命令写入 aof_buf 后调用 write 操作，write 操作完成线程返回。fsync 同步文件由专门线程每秒调用一次，在性能和持久化方面做了很好的折中
3. appendfsync no //命令写入 aof_buf 后调用系统 write 操作，不对 AOF 文件做 fsync 同步，完全依赖 os，性能最好,持久化没保证

##### AOF重写

aof 的方式也同时带来了另一个问题。持久化文件会变的越来越大。AOF重写能压缩文件体积，有以下原因：

+ 多条命令合并。例如我们调用 incr test 命令 100 次，文件中必须保存全部的 100 条命令，其实有 99 条都是多余的。因为要恢复数据库的状态其实文件中保存一条 set test 100 就够了。
+ 进程内已经超时的数据不会再写入文件
+ 旧的AOF文件包含无效命令，如 del key1,hdel key2,srem keys,set alll, set a222等。重写使用进程内数据直接生成，这样新的AOF文件只保留最终数据的写入命令。

AOF重写降低了文件占用空间，而且也可以更快地被 Redis 加载。为了压缩 aof 的持久化文件， redis 提供了两种方式：
1. 手动触发：`bgrewriteaof` 命令。收到此命令 redis 将使用与快照类似的方式将内存中的数据以命令的方式保存到临时文件中，最后替换原来的文件。
2. 自动触发：根据`auto-aof-rewrite-min-size`和`auto-aof-rewrite-percentage`参数确定自动触发时机。
   + auto-aof-rewrite-min-size：表示运行AOF重写时文件最小体积，默认为64MB。
   + auto-aof-rewrite-percentage：代表当前AOF文件空间（aof_current_size）和上一次重写后AOF文件空间（aof_base_size）的比值。
   自动触发时机=aof_current_size > auto-aof-rewrite-minsize &&（aof_current_size-aof_base_size）/aof_base_size >= auto-aof-rewritepercentage  
   其中aof_current_size和aof_base_size可以在info Persistence统计信息中查看。

### Redis 复制

在分布式系统中为了解决单点问题，通常会把数据复制多个副本部署到其他机器，满足故障恢复和负载均衡等需求。Redis也是如此，它为我们提供了复制功能，实现了相同数据的多个Redis副本。复制功能是高可用Redis的基础，后面章节的哨兵和集群都是在复制的基础上实现高可用的。

参与复制的Redis实例划分为主节点（master）和从节点（slave）。默认情况下，Redis都是主节点。每个从节点只能有一个主节点，而主节点可以同时具有多个从节点。复制的数据流是单向的，只能由主节点复制到从节点。配置复制的方式有以下三种：

1. 在配置文件中加入 `slaveof {masterHost} {masterPort}` 随Redis启动生效。
2. 在 redis-server 启动命令后加入 `--slaveof {masterHost} {masterPort}` 选项，启动后生效。
3. 直接使用命令：`slaveof {masterHost} {masterPort}` ，命令执行后生效。

主从节点复制成功建立后，可以使用 `info replication` 命令查看复制相关状态。

slaveof命令不但可以建立复制，还可以在从节点执行 `slaveof no one` 来断开与主节点复制关系。从节点断开复制后并不会抛弃原有数据，只是无法再获取主节点上的数据变化。把当前从节点对主节点的复制切换到另一个主节点，执行 `slaveof {newMasterIp} {newMasterPort}` 命令即可，切主后从节点会清空之前所有的数据，线上人工操作时小心 slaveof 在错误的节点上执行或者指向错误的主节点。

### Redis 分布式集群

Redis 支持一主多从的主从复制和集群分片的组合模式。

#### Redis 集群的数据分片
Redis 集群没有使用一致性hash, 而是引入了哈希槽的概念.

Redis 集群有16384个哈希槽,每个key通过CRC16校验后对16384取模来决定放置哪个槽.集群的每个节点负责一部分hash槽,举个例子,比如当前集群有3个节点,那么:

节点 A 包含 0 到 5500号哈希槽.
节点 B 包含5501 到 11000 号哈希槽.
节点 C 包含11001 到 16384号哈希槽.
这种结构很容易添加或者删除节点. 比如如果我想新添加个节点D, 我需要从节点 A, B, C中得部分槽到D上. 如果我想移除节点A,需要将A中的槽移到B和C节点上,然后将没有任何槽的A节点从集群中移除即可. 由于从一个节点将哈希槽移动到另一个节点并不会停止服务,所以无论添加删除或者改变某个节点的哈希槽的数量都不会造成集群不可用的状态.

#### Redis 集群的主从复制模型
为了使在部分节点失败或者大部分节点无法通信的情况下集群仍然可用，所以集群使用了主从复制模型,每个主节点都会有N-1个复制品.

**在我们例子中具有A，B，C三个节点的集群,在没有复制模型的情况下,如果节点B失败了，那么整个集群就会以为缺少5501-11000这个范围的槽而不可用。**
然而如果在集群创建的时候（或者过一段时间）我们为每个节点添加一个从节点A1，B1，C1,那么整个集群便有三个master节点和三个slave节点组成，这样在节点B失败后，集群便会选举B1为新的主节点继续服务，整个集群便不会因为槽找不到而不可用了

不过当B和B1 都失败后，集群是不可用的.

#### 搭建集群

1. 准备节点

   Redis集群一般由多个节点组成，节点数量至少为6个才能保证组成完整高可用的集群。每个节点需要开启配置 cluster-enabled yes，让Redis运行在集群模式下。建议为集群内所有节点统一目录，一般划分三个目录：conf、data、log，分别存放配置、数据和日志相关文件。把6个节点配置统一放在conf目录下，命名规则为 redis-port.conf ,集群相关配置如下：

   ```
   bind 127.0.0.1
   port 6379
   tcp-backlog 511
   loglevel notice
   logfile "log/node-6379.log"
   cluster-enabled yes
   \# 节点超时时间，单位毫秒
   cluster-node-timeout 15000
   \# 集群内部配置文件
   cluster-config-file "nodes-6379.conf"
   ```

   配置文件创建好后，分别启动6个实例：

   `redis-server conf\redis-6379.conf`

   实例启动后，可以用 redis-cli 连接到任意一台机器，然后执行：`cluster nodes`，会发现只有一台机器，因为此时6台实例之间并不知道对方。

2. 节点握手

   节点握手是指一批运行在集群模式下的节点通过Gossip协议彼此通信，达到感知对方的过程。节点握手是集群彼此通信的第一步，由客户端发起命令：`cluster meet {ip} {port}`

   分别执行上述命令将6台机器加入到集群后，集群还不能正常工作，这时集群处于下线状态，所有的数据读写都被禁止，可以使用 `cluster info` 命令查看集群的当前状态：

   ```
   cluster_state:fail
   cluster_slots_assigned:0
   cluster_slots_ok:0
   cluster_slots_pfail:0
   cluster_slots_fail:0
   cluster_known_nodes:6
   cluster_size:0
   cluster_current_epoch:1
   cluster_my_epoch:1
   cluster_stats_messages_ping_sent:101
   cluster_stats_messages_meet_sent:5
   cluster_stats_messages_sent:106
   cluster_stats_messages_pong_received:106
   cluster_stats_messages_received:106
   ```

   因为此时还没有为各个节点分配槽，所以现在集群还是不可用的。

3. 分配槽

   Redis集群把所有的数据映射到16384个槽中。每个key会映射为一个固定的槽，只有当节点分配了槽，才能响应和这些槽关联的键命令。通过 `cluster addslots` 命令为节点分配槽。这里利用bash特性批量设置槽（slots），命令如下：

   ```
   redis-cli -h 127.0.0.1 -p 6379 cluster addslots {0...5461}
   windows:FOR /L %i IN (0,1,5461) DO ( redis-cli.exe -h 127.0.0.1 -p 6379 CLUSTER ADDSLOTS %i )
   redis-cli -h 127.0.0.1 -p 6380 cluster addslots {5462...10922}
   windows:FOR /L %i IN (5462,1,10922) DO ( redis-cli.exe -h 127.0.0.1 -p 6380 CLUSTER ADDSLOTS %i )
   redis-cli -h 127.0.0.1 -p 6381 cluster addslots {10923...16383}
   windows:FOR /L %i IN (10923,1,16383) DO ( redis-cli.exe -h 127.0.0.1 -p 6381 CLUSTER ADDSLOTS %i )
   ```
   
   分配好槽以后，整个集群就是可用的了：
   
   ```
   127.0.0.1:6379> cluster info
   cluster_state:ok
   cluster_slots_assigned:16384
   cluster_slots_ok:16384
   cluster_slots_pfail:0
   cluster_slots_fail:0
   cluster_known_nodes:6
   cluster_size:3
   cluster_current_epoch:1
   cluster_my_epoch:1
   cluster_stats_messages_ping_sent:10253
   cluster_stats_messages_sent:10253
   cluster_stats_messages_pong_received:10017
   cluster_stats_messages_received:10017
   ```
   
   目前还有三个节点没有使用，作为一个完整的集群，每个负责处理槽的节点应该具有从节点，保证当它出现故障时可以自动进行故障转移。集群模式下，Reids节点角色分为主节点和从节点。首次启动的节点和被分配槽的节点都是主节点，从节点负责复制主节点槽信息和相关的数据。使用`cluster replicate {nodeId}`命令让一个节点成为从节点。其中命令执行必须在对应的从节点上执行，`nodeId` 是要复制主节点的节点ID，命令如下：
   
   ```
   redis-cli -h 127.0.0.1 -p 6382 cluster replicate a62b0061a541872d5c41e75efe987283aed167f6
   redis-cli -h 127.0.0.1 -p 6383 cluster replicate a935e92708b09b6ad2f4ae10c433be519c4ecfd0
   redis-cli -h 127.0.0.1 -p 6384 cluster replicate c94dde54d0f6aa124356d8a58c6be0a8c4ae8058
   ```
   
   执行完后，就会发现此时的集群是3主3从的集群了：
   
   ```
   127.0.0.1:6379> cluster nodes
   55ab7f5801001c9c71809a86b394f4fd029e87a2 127.0.0.1:6382@16382 slave a62b0061a541872d5c41e75efe987283aed167f6 0 1612365584000 4 connected
   a9f663f870d15f1d2c13665d49e4b8a877ce7467 127.0.0.1:6384@16384 slave c94dde54d0f6aa124356d8a58c6be0a8c4ae8058 0 1612365582000 5 connected
   def8275ed31a1542892ff3444153476484f3934c 127.0.0.1:6383@16383 slave a935e92708b09b6ad2f4ae10c433be519c4ecfd0 0 1612365584293 2 connected
   a62b0061a541872d5c41e75efe987283aed167f6 127.0.0.1:6379@16379 myself,master - 0 1612365583000 1 connected 0-5460
   c94dde54d0f6aa124356d8a58c6be0a8c4ae8058 127.0.0.1:6381@16381 master - 0 1612365585386 3 connected 10923-16383
   a935e92708b09b6ad2f4ae10c433be519c4ecfd0 127.0.0.1:6380@16380 master - 0 1612365583000 2 connected 5461-10922
   ```
   
   使用 `redis-cli -c` 参数连接到集群中任意一台机器上，然后使用 `get/set` 命令存取数据，这样会用 key 计算 hash 然后算出对应的槽，客户端也会自动重定向到槽所对应的节点上存取数据。
   `redis-cli -c --cluster call 127.0.0.1:6379 keys *` 查看集群中的所有key

#### Redis 一致性保证
Redis 并不能保证数据的强一致性. 这意味这在实际中集群在特定的条件下可能会丢失写操作.

第一个原因是因为集群是用了异步复制. 写操作过程:

客户端向主节点B写入一条命令.
主节点B向客户端回复命令状态.
主节点将写操作复制给他得从节点 B1, B2 和 B3.
主节点对命令的复制工作发生在返回命令回复之后， 因为如果每次处理命令请求都需要等待复制操作完成的话， 那么主节点处理命令请求的速度将极大地降低 —— 我们必须在性能和一致性之间做出权衡。 注意：Redis 集群可能会在将来提供同步写的方法。 
Redis 集群另外一种可能会丢失命令的情况是集群出现了网络分区， 并且一个客户端与至少包括一个主节点在内的少数实例被孤立。

举个例子 假设集群包含 A 、 B 、 C 、 A1 、 B1 、 C1 六个节点， 其中 A 、B 、C 为主节点， A1 、B1 、C1 为A，B，C的从节点， 还有一个客户端 Z1 假设集群中发生网络分区，那么集群可能会分为两方，大部分的一方包含节点 A 、C 、A1 、B1 和 C1 ，小部分的一方则包含节点 B 和客户端 Z1 .

Z1仍然能够向主节点B中写入, 如果网络分区发生时间较短,那么集群将会继续正常运作,如果分区的时间足够让大部分的一方将B1选举为新的master，那么Z1写入B中得数据便丢失了.

注意， 在网络分裂出现期间， 客户端 Z1 可以向主节点 B 发送写命令的最大时间是有限制的， 这一时间限制称为节点超时时间（node timeout）， 是 Redis 集群的一个重要的配置选项：
