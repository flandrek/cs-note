## Redis：Remote Dictionary Server

redis 是一个 C 语言开发的 key-value 类型的内存数据库，整个数据库全部加载在内存中执行，定期通过**异步操作**将数据 flush 到硬盘中保存。

单个 value 的最大值为 1GB，key 的最大值为 512MB ，String 类型的 value 最大为 512 M。

Redis 支持 String，LIst，Set，SortedSet，Hashes。

**用途：**

1. 可用 List 作为 FIFO 双向链表，实现高性能的消息队列；
2. 用 Set 实现高性能的 tag 系统；

#### Redis 与 memcached 的对比

1. memcached 只支持字符串，Redis 支持更多数据类型；
2. memcached 单个 item 最大存储为 1M，Redis 最大值为 1G，存储容量更大；
3. memcached 不支持数据的持久化，Redis 支持；
4. Redis 在很多方面具备数据库的特征，或者说就是一个数据库系统，而Memcache只是简单的 K / V 缓存；
5. Redis 读取效率更高，且支持事务，操作都是原子性的；
6. 他们的扩展都需要做集群，实现方式为 master-slave、Hash。

#### Redis 单线程

指 redis 在处理多个网络请求时，使用的是同一个线程。当有请求过来时，redis 把线程唤醒，去轮询这些流，再按照顺序去处理这些请求，通过 redis-client 的链表。

在处理请求时使用一个线程，持久化和其它操作，通过 fork 子线程实现。

Redis 使用单线程的 IO 复用，自己封装了一个 AeEvent 事件处理框架，实现了 epoll，select...，对于单纯的 IO 操作，单线程可以将速度优势发挥到最大。

**单线程为什么这么快？**

1. 纯内存操作；
2. 非阻塞 IO ，即实现了多路复用；
3. 没有线程切换的资源消耗；

#### Redis key的过期策略



## Redis 集群方案

## Redis集群实现方式

### 实现基础——分区

- 分区是分割数据到多个Redis实例的处理过程，因此每个实例只保存 key 的一个子集。
- 通过利用多台计算机内存的和值，允许我们构造更大的数据库。
- 通过多核和多台计算机，允许我们扩展计算能力；通过多台计算机和网络适配器，允许我们扩展网络带宽。

### 集群的几种实现方式

- 客户端分片
- 基于代理的分片
- 路由查询

### 客户端分片

- 由客户端决定 key 写入或者读取的节点。
- 包括 jedis 在内的一些客户端，实现了客户端分片机制。

原理如下所示：

 

![img](E:\markdown笔记\图片\1521743-5490a7b34deae42c.png)

 

**特性**

- 优点
  - 简单，性能高。
- 缺点
  - 业务逻辑与数据存储逻辑耦合
  - 可运维性差
  - 多业务各自使用 redis，集群资源难以管理
  - 不支持动态增删节点

### 基于代理的分片

- 客户端发送请求到一个代理，代理解析客户端的数据，将请求转发至正确的节点，然后将结果回复给客户端。
- 开源方案
  - Twemproxy
  - codis

基本原理如下所示：

 

![img](E:\markdown笔记\图片\1521743-5b8e35bb04543259.png)

 

 

**特性**

- 透明接入Proxy 的逻辑和存储的逻辑是隔离的。
- 业务程序不用关心后端Redis实例，切换成本低。
- 代理层多了一次转发，性能有所损耗。

### 路由查询

- 将请求发送到任意节点，接收到请求的节点会将查询请求发送到正确的节点上执行。
- 开源方案
  - Redis-cluster

基本原理如下所示：

 

![img](E:\markdown笔记\图片\1521743-204c4cc057006efa.png)

 

### 集群的挑战

- 涉及多个 key 的操作通常是不被支持的。涉及多个 key 的 redis 事务不能使用。
  - 举例来说，当两个 set 映射到不同的 redis 实例上时，你就不能对这两个 set 执行交集操作。
- 不能保证集群内的数据均衡。
  - 分区的粒度是 key，如果某个 key 的值是巨大的 set、list，无法进行拆分。
- 增加或删除容量也比较复杂。
  - redis集群需要支持在运行时增加、删除节点的透明数据平衡的能力。



## Redis集群各种方案原理

### Twemproxy

- Proxy-based
- twitter 开源，C语言编写，单线程。
- 支持 Redis 或 Memcached 作为后端存储。

**Twemproxy高可用部署架构**

 

![img](E:\markdown笔记\图片\1521743-4dc3262674558d8f.png)

 

#### Twemproxy特性

- 支持失败节点自动删除
  - 与 redis 的长连接，连接复用，连接数可配置
- 自动分片到后端多个 redis 实例上支持 redis pipelining 操作
  - 多种hash算法：能够使用不同的分片策略和散列函数
  - 支持一致性hash，但是使用DHT之后，从集群中摘除节点时，不会进行rehash操作
  - 可以设置后端实例的权重
- 支持状态监控
- 支持select切换数据库

> redis的管道(Pipelining)操作是一种异步的访问模式，一次发送多个指令，不同步等待其返回结果。这样可以取得非常好的执行效率。调用方法如下：
>
> ```
> `@Test``public` `void` `testPipelined() {``    ``Jedis jedis = ``new` `Jedis(``"localhost"``);``    ``Pipeline pipeline = jedis.pipelined();``    ``long` `start = System.currentTimeMillis();``    ``for` `(``int` `i = ``0``; i < ``100000``; i++) {``        ``pipeline.set(``"p"` `+ i, ``"p"` `+ i);``    ``}``    ``List<Object> results = pipeline.syncAndReturnAll();``    ``long` `end = System.currentTimeMillis();``    ``System.out.println(``"Pipelined SET: "` `+ ((end - start)/``1000.0``) + ``" seconds"``);``    ``jedis.disconnect();``}`
> ```

#### Twemproxy不足

- 性能低：代理层损耗 && 本身效率低下
- Redis功能支持不完善
  - 不支持针对多个值的操作，比如取sets的子交并补等（MGET 和 DEL 除外）
  - 不支持Redis的事务操作
  - 出错提示还不够完善
- 集群功能不够完善
  - 仅作为代理层使用
  - 本身不提供动态扩容，透明数据迁移等功能
- 失去维护
  - 最近一次提交在一年之前。Twitter 内部已经不再使用。



### Redis Cluster

- Redis官网推出，可线性扩展到1000个节点
- 无中心架构
- 一致性哈希思想
- 客户端直连redis服务，免去了 proxy 代理的损耗

#### Redis Cluster模型

 

![img](E:\markdown笔记\图片\1521743-158efb87b969a0a8.png)

 

#### Redis-cluster原理

- Hash slot。集群内的每个 redis 实例监听两个 tcp 端口，6379（默认）用于服务客户端查询，16379（默认服务端口 + 10000）用于集群内部通信。
  - key 的空间被分到 16384 个 hash slot 里；
  - 计算 key 属于哪个slot，CRC16(key) & 16384。
- 节点间状态同步：gossip 协议，最终一致性。节点间通信使用轻量的二进制协议，减少带宽占用。

 

> **Gossip 算法**又被称为反熵（Anti-Entropy），熵是物理学上的一个概念，代表杂乱无章，而反熵就是在杂乱无章中寻求一致，这充分说明了 Gossip 的特点：在一个有界网络中，每个节点都随机地与其他节点通信，经过一番杂乱无章的通信，最终所有节点的状态都会达成一致。每个节点可能知道所有其他节点，也可能仅知道几个邻居节点，只要这些节可以通过网络连通，最终他们的状态都是一致的，当然这也是疫情传播的特点。
> 简单的描述下这个协议，首先要传播谣言就要有种子节点。种子节点每秒都会随机向其他节点发送自己所拥有的节点列表，以及需要传播的消息。任何新加入的节点，就在这种传播方式下很快地被全网所知道。这个协议的神奇就在于它从设计开始就没想到信息一定要传递给所有的节点，但是随着时间的增长，在最终的某一时刻，全网会得到相同的信息。当然这个时刻可能仅仅存在于理论，永远不可达。

 

#### Redis-cluster请求路由方式

- Redis-Cluster 借助客户端实现了混合形式的路由查询
  查询路由并非直接从一个redis节点到另外一个redis，而是借助客户端转发到正确的节点。根据客户端的实现方式，可以分为以下两种：包括 Jedis 在内的许多 redis client，已经实现了对 Redis Cluster 的支持。
  - dummy client
  - smart client

查询路由的流程如下所示：

 

![img](E:\markdown笔记\图片\1521743-65d69d2217bef71f.png)

 

> Redis cluster采用这种架构的考虑：
>
> - 减少redis实现的复杂度
> - 降低客户端等待的时间。Smart client 可以在客户端缓存 slot 与 redis 节点的映射关系，当接收到 MOVED 响应时，会修改缓存中的映射关系。请求时会直接发送到正确的节点上，减少一次交互。

#### Redis Cluster特性

- 高性能
- 支持动态扩容，对业务透明
- 具备 Sentinel 的监控和自动 Failover 能力

#### Redis Cluster不足

- 官方未提供图形管理工具，运维比较复杂要求客户端必须支持 cluster 协议
  - 比如数据迁移，纯手动操作，先分配 slot，再将 slot 中对应的 key 迁移至新的节点。
- Redis命令支持不完整
  - 对于批量的指令，如 mget 支持不完整；不支持事务；不支持数据库切换，只能使用0号数据库……
- 集群管理与数据存储耦合
  - 比如如果集群管理有 bug，需要升级整个 redis。

### Codis

- 豌豆荚开源的 proxy-based 的 redis 集群方案
- 支持透明的扩/缩容
- 运维友好同类产品中最全面的 redis 命令支持
  - GUI 监控界面和管理工具

#### Codis部署拓扑

 

![img](E:\markdown笔记\图片\1521743-1ffebc4d3b1cd6c4.png)

 

#### Codis数据存储

- 数据根据key分布到 1024 个 slot 内
- Key 的 slotid 计算方法 crc32(key) % 1024
- 每个 codis-server 负责一部分数据
- 比如后端有3组 codis-server，每个 server 负责的 slot 范围可以这样设置：
  - group0 0-300
  - group1 301-600
  - group2 601-1023

#### Codis模块简介

- Codis Server
  - 基于 redis-2.8.21 分支开发。增加了额外的数据结构，以支持 slot 有关的操作以及数据迁移指令。
- Codis Proxy
  - 客户端连接的 Redis 代理服务, 实现了 Redis 协议。
  - 对于同一个业务集群而言，可以同时部署多个 codis-proxy 实例
  - 不同 codis-proxy 之间由 codis-dashboard 保证状态同步。
- Codis Dashboard
  - 集群管理工具，支持 codis-proxy、codis-server 的添加、删除，以及据迁移等操作。
  - 所有对集群的修改都必须通过 codis-dashboard 完成。
- Codis Admin
  - 集群管理的命令行工具。
  - 可用于控制 codis-proxy、codis-dashboard 状态以及访问外部存储。
- Codis FE
  - 集群管理界面。
- Codis HA
  - 为集群提供高可用。
  - 依赖 codis-dashboard 实例，自动抓取集群各个组件的状态；
  - 会根据当前集群状态自动生成主从切换策略，并在需要时通过 codis-dashboard 完成主从切换。
- Storage
  - 为集群状态提供外部存储。
  - 目前仅提供了 Zookeeper 和 Etcd 两种实现，但是提供了抽象的 interface 可自行扩展。

#### Codis主从切换

- Codis-HA：自动切换主从的工具。
- 通过RESTful API从 codis-dashboard 中拉取集群状态
- 默认以 5s 为周期
- 该工具会在检测到 master 挂掉的时候主动应用主从切换策略，向codis-dashboard发送主从切换命令。
- 对自动主从切换的支持比较弱
  - 主从切换的限制较多，必须在系统其他组件状态健康，且距离上次主从同步数据在一定时间间隔内才可以执行。
  - 多slave场景，提升其中的一个slave为master，其他的slave仍然会从旧的master同步数据，需要管理员手动操作。

#### Codis数据迁移流程

- 前提：Codis单条数据迁移是原子操作
- 单条数据迁移过程：
  - 随机选取指定 slot 中的一个 <K,V>
  - 将这个<K,V>传输给另外一个 codis-server
  - 传输成功后，把本地的这个 <K,V>删除

迁移过程如下所示：

 

![img](E:\markdown笔记\图片\1521743-f6a71426e11d04a0.png)

 

迁移过程中，Codis-dashboard与proxy通过zk通信，路由表信息全部存放在zk，保证所有proxy的视图一致。
Codis如何保证数据迁移过程的正确及透明？

- codis-config 在实际修改slot状态之前，会确保所有的 proxy 收到这个迁移请求。
- 迁移过程中, 如果客户端请求 slot 的 key 数据,proxy 会将请求转发到group2上。
  - proxy会先在group1上强行执行一次 MIGRATE key 将这个键值提前迁移过来，
  - 然后再到group2上正常读取

### Codis VS Redis

| 对比参数       | Codis                                                        | Redis-cluster                                   |
| -------------- | ------------------------------------------------------------ | ----------------------------------------------- |
| Redis版本      | 基于2.8分支开发                                              | >= 3.0                                          |
| 部署           | 较复杂。                                                     | 简单                                            |
| 运维           | Dashboard,运维方便。                                         | 运维人员手动通过命令操作。                      |
| 监控           | 可在Dashboard里监控当前redis-server节点情况，较为便捷。      | 不提供监控功能。                                |
| 组织架构       | Proxy-Based, 类中心化架构，集群管理层与存储层解耦。          | P2P模型，gossip协议负责集群内部通信。去中心化   |
| 伸缩性         | 支持动态伸缩。                                               | 支持动态伸缩                                    |
| 主节点失效处理 | 自动选主。                                                   | 自动选主。                                      |
| 数据迁移       | 简单。支持透明迁移。                                         | 需要运维人员手动操作。支持透明迁移。            |
| 升级           | 基于redis 2.8分支开发，后续升级不能保证；Redis-server必须是此版本的codis，无法使用新版本redis的增强特性。 | Redis官方推出，后续升级可保证。                 |
| 可靠性         | 经过线上服务验证，可靠性较高。                               | 新推出，坑会比较多。遇到bug之后需要等官网升级。 |

> 1. 理论上，redis-cluster的性能更高，单次请求的延时低。另外，经过实测，两种架构后端单台redis-server的条件下，TPS基本没有差别。
> 2. Codis的高可用依赖jodis，或者使用LVS进行高可用部署。

### 选择Codis的原因

- Redis-cluster 没有成熟的应用案例
- Codis支持的命令更加丰富
- 迁移至redis cluster需要修改现有实现；而迁移到codis没这么麻烦，只要使用的redis命令在codis支持的范围之内，只要修改一下配置即可接入。
- Codis提供的运维工具更加友好

## Codis简单压测

### 单台Codis Server压测结果

 

![img](E:\markdown笔记\图片\1521743-f3986f79bcf5c6b4.png)

 

### 三台Codis-Server的集群压测结果

 

![img](E:\markdown笔记\图片\1521743-e66650f026f9c004.png)

 

**压测结论**

- 针对codis集群压测过程中，后端codis server平均CPU占用约为70%~80%。
- 单台codis的TPS在6w~7w左右，集群的TPS在17w左右，可以达到近乎线性扩展的能力
- 测试期间后台codis-server的负载没有跑满，仍然具有继续压测的潜力。
- 测试期间proxy机器负载低，可以支撑更多后端codis server实例。

## 使用codis注意事项

- 不支持的命令
  - [不支持的命令列表](https://link.jianshu.com/?t=https://github.com/CodisLabs/codis/blob/3.0.3/doc/unsupported_cmds.md)
- 防止key冲突
  - 建议现有业务接入codis时，加入项目前缀以作区分。
- 使用spring data redis操作codis时，只能使用RedisTemplate系列接口，Cache系列接口不可用
  - Cache系列的接口使用了redis的事务指令，事务在Codis以及Redis Cluster中均未提供支持。
- 避免key value巨大的数据
  - 吞吐量提升不明显
  - 可能造成各codis实例资源占用不均衡





# 三张图秒懂Redis集群设计原理

*Redis*集群设计包括*2*部分：哈希*Slot*和节点主从，本篇博文通过*3*张图来搞明白*Redis*的集群设计。

## 节点主从

主从设计不算什么新鲜玩意，在数据库中我们也经常用主从来做读写分离，直接上图：

*![img](E:\markdown笔记\图片\20171108222727512.png)*



图上能看得到的信息：

*1，* 只有*1*个*Master*，可以有*N*个*slaver*，而且*Slaver*也可以有自己的*Slaver*，由于这种主从的关系决定他们是在配置阶段就要指定他们的上下级关系，而不是*Zookeeper*那种平行关系是自主推优出来的。

*2，* 读写分离，*Master*只负责写和同步数据给*Slaver*，*Slaver*承担了被读的任务，所以*Slaver*的扩容只能提高读效率不能提高写效率。

*3，* *Slaver*先将*Master*那边获取到的信息压入磁盘，再*load*进内存，*client*端是从内存中读取信息的，所以*Redis*是内存数据库。

当一个新的*Slaver*加入到这个集群时，会主动找*Master*来拜码头，*Master*发现新的小弟后将全量数据发送给新的*Slaver*，数据量越大性能消耗也就越大，所以尽量避免在运行时做*Slaver*的扩容。

简单总结下主从模式的设计：

优点：读写分离，通过增加*Slaver*可以提高并发读的能力。

缺点：*Master*写能力是瓶颈。

​          虽然理论上对*Slaver*没有限制但是维护*Slaver*开销总将会变成瓶颈。

​          *Master*的*Disk*大小也将会成为整个*Redis*集群存储容量的瓶颈。

## 哈希*Slot*

这个艺名看起来很文艺，但也不是什么新技术，他的真名就叫分表分库，再上一个图：





![img](E:\markdown笔记\图片\20171113151049211.png)



图上能看到的信息：

*1，* 对象保存到*Redis*之前先经过*CRC16*哈希到一个指定的*Node*上，例如*Object4*最终*Hash*到了*Node1*上。

*2，* 每个*Node*被平均分配了一个*Slot*段，对应着*0-16384*，*Slot*不能重复也不能缺失，否则会导致对象重复存储或无法存储。

*3，* *Node*之间也互相监听，一旦有*Node*退出或者加入，会按照*Slot*为单位做数据的迁移。例如*Node1*如果掉线了，*0-5640*这些*Slot*将会平均分摊到*Node2*和*Node3*上*,*由于*Node2*和*Node3*本身维护的*Slot*还会在自己身上不会被重新分配，所以迁移过程中不会影响到*5641-16384Slot*段的使用。

简单总结下哈希*Slot*的优缺点：

缺点：每个*Node*承担着互相监听、高并发数据写入、高并发数据读出，工作任务繁重

优点：将*Redis*的写操作分摊到了多个节点上，提高写的并发能力，扩容简单。

 

## 双剑合并

看到这里大家也就发现了，主从和哈希的设计优缺点正好是相互弥补的，将图一每一套主从对应到图二中的每一个*Node*，就是*Redis*集群的终极形态，先*Hash*分逻辑节点，然后每个逻辑节点内部是主从，如图：

![img](E:\markdown笔记\图片\20171113151113823.png)

想扩展并发读就添加*Slaver*，想扩展并发写就添加*Master*，想扩容也就是添加*Master*，任何一个*Slaver*或者几个*Master*挂了都不会是灾难性的故障。





# Redis Cluster探索与思考

## Redis Cluster的基本原理和架构

Redis Cluster 是分布式 Redis 的实现。随着 Redis 版本的更替，以及各种已知 bug 的 fixed，在稳定性和高可用性上有了很大的提升和进步，越来越多的企业将 Redis Cluster 实际应用到线上业务中，通过从社区获取到反馈社区的迭代，为 Redis Cluster 成为一个可靠的企业级开源产品，在简化业务架构和业务逻辑方面都起着积极重要的作用。下面从 Redis Cluster 的基本原理为起点开启 Redis Cluster 在业界的分析与思考之旅。

### 基本原理

Redis Cluster 的基本原理可以从数据分片、数据迁移、集群通讯、故障检测以及故障转移等方面进行了解，Cluster相关的代码也不是很多，注释也很详细，可自行查看，地址是：<https://github.com/antirez/redis/blob/unstable/src/cluster.c>。这里由于篇幅的原因，主要从数据分片和数据迁移两方面进行详细介绍：

**数据分片**

Redis Cluster 在设计中没有使用一致性哈希（Consistency Hashing），而是使用数据分片（Sharding）引入哈希槽（hash slot）来实现；一个 Redis Cluster 包含 16384（0~16383）个哈希槽，存储在 Redis Cluster 中的所有键都会被映射到这些 slot 中，集群中的每个键都属于这 16384个哈希槽中的一个，集群使用公式slot=CRC16（key）/ 16384来计算 key 属于哪个槽，其中 CRC16(key) 语句用于计算 key 的 CRC16 校验和。

集群中的每个主节点（Master）都负责处理16384个哈希槽中的一部分，当集群处于稳定状态时，每个哈希槽都只由一个主节点进行处理，每个主节点可以有一个到 N 个从节点（Slave），当主节点出现宕机或网络断线等不可用时，从节点能自动提升为主节点进行处理。

如图1，ClusterNode 数据结构中的 slots 和 numslots 属性记录了节点负责处理哪些槽。其中，slot 属性是一个**二进制位数组（bitarray）**，其长度为 16384 / 8 = 2048 Byte，共包含 16384 个二进制位。集群中的 Master 节点用 bit（0和1）来标识对于某个槽是否拥有。比如，对于编号为1的槽，Master 只要判断序列第二位（索引从0开始）的值是不是 1 即可，时间复杂度为O(1)。

![图1  ClusterNode 数据结构](E:\markdown笔记\图片\58db1a6090500.jpg)



图1 ClusterNode 数据结构

![img](E:\markdown笔记\图片\58db195cd17e9.jpg)

集群中所有槽的分配信息都保存在 ClusterState 数据结构的 slots 数组中，程序要检查槽 i 是否已经被分配或者找出处理槽 i 的节点，只需要访问 clusterState.slots[i] 的值即可，复杂度也为O(1)。ClusterState 数据结构如图2所示。

![图2  ClusterState数据结构](E:\markdown笔记\图片\58db1b12b188a.jpg)



图2 ClusterState数据结构

查找关系如图3所示。

![图3  查找关系图](E:\markdown笔记\图片\58db1b3d91743.jpg)



图3 查找关系图



**数据迁移**

数据迁移可以理解为 slot 和 key 的迁移，这个功能很重要，极大地方便了集群做线性扩展，以及实现平滑的扩容或缩容。那么它是一个怎样的实现过程？下面举个例子：现在要将 Master A 节点中编号为 1、2、3 的 slot 迁移到 Master B 节点中，在 slot 迁移的中间状态下，slot 1、2、3 在 Master A 节点的状态表现为 MIGRATING ,在Master B 节点的状态表现为 IMPORTING。

**MIGRATING状态**

这个状态如图4所示是被迁移 slot 在当前所在 Master A 节点中出现的一种状态，预备迁移 slot 从 Mater A 到Master B 的时候，被迁移 slot 的状态首先变为 MIGRATING 状态，当客户端请求的某个 key 所属的 slot 的状态处于 MIGRATING 状态时，会出现以下几种情况：

![图4  slot迁移的中间状态](E:\markdown笔记\图片\58db1c24c5b58.jpg)



图4 slot迁移的中间状态



- 如果 key 存在则成功处理。

- 如果 key 不存在，则返回客户端 ASK，客户端根据 ASK 首先发送 ASKING 命令到目标节点，然后发送请求的命令到目标节点。

- 当 key 包含多个命令时：

  - 如果都存在则成功处理
  - 如果都不存在，则返回客户端 ASK
  - 如果一部分存在，则返回客户端 TRYAGAIN，通知客户端稍后重试，这样当所有的 key 都迁移完毕，客户端重试请求时会得到 ASK，然后经过一次重定向就可以获取这批键

- 此时并不刷新客户端中 node 的映射关系

**IMPORTING状态**

这个状态如图2所示是被迁移 slot 在目标 Master B 节点中出现的一种状态，预备迁移 slot 从 Mater A 到 Master B 的时候，被迁移 slot 的状态首先变为 IMPORTING 状态。在这种状态下的 slot 对客户端的请求可能会有下面几种影响：

- 如果key不存在则新建。
- 如果key不在该节点上，命令会被MOVED重定向，刷新客户端中node的映射关系。
- 如果是ASKING命令则命令会被执行，从而key没在被迁移的节点，已经被迁移到目标节点的情况命令可以被顺利执行。

**键空间迁移**

这是完成数据迁移的重要一步，键空间迁移是指当满足了slot迁移前提的情况下，通过相关命令将slot 1、2、3中的键空间从Master A节点转移到Master B节点，这个过程由MIGRATE命令经过3步真正完成数据转移。步骤示意如图5。

![图5  表空间迁移步骤](E:\markdown笔记\图片\58db1ecb8d22d.jpg)



图5 表空间迁移步骤



经过上面三步可以完成键空间数据迁移，然后再将处于MIGRATING和IMPORTING状态的槽变为常态即可，从而完成整个重新分片的过程。

### 架构

实现细节：

- Redis Cluster中节点负责存储数据，记录集群状态，集群节点能自动发现其他节点，检测出节点的状态，并在需要时剔除故障节点，提升新的主节点。
- Redis Cluster中所有节点通过PING-PONG机制彼此互联，使用一个二级制协议(Cluster Bus) 进行通信，优化传输速度和带宽。发现新的节点、发送PING包、特定情况下发送集群消息，集群连接能够发布与订阅消息。
- 客户端和集群中的节点直连，不需要中间的Proxy层。理论上而言，客户端可以自由地向集群中的所有节点发送请求，但是每次不需要连接集群中的所有节点，只需要连接集群中任何一个可用节点即可。当客户端发起请求后，接收到重定向（MOVED\ASK）错误，会自动重定向到其他节点，所以客户端无需保存集群状态。不过客户端可以缓存键值和节点之间的映射关系，这样能明显提高命令执行的效率。
- Redis Cluster中节点之间使用**异步复制**，在分区过程中存在窗口，容易导致丢失写入的数据，集群即使努力尝试所有写入，但是以下两种情况可能丢失数据：
  - 命令操作已经到达主节点，但在主节点回复的时候，写入可能还没有通过主节点复制到从节点那里。如果这时主节点宕机了，这条命令将永久丢失。以防主节点长时间不可达而它的一个从节点已经被提升为主节点。
  - 分区导致一个主节点不可达，然而集群发送**故障转移(failover)**，提升从节点为主节点，原来的主节点再次恢复。一个没有更新路由表（routing table）的客户端或许会在集群把这个主节点变成一个从节点（新主节点的从节点）之前对它进行写入操作，导致数据彻底丢失。
- Redis集群的节点不可用后，在经过集群半数以上Master节点与故障节点通信超过cluster-node-timeout时间后，认为该节点故障，从而集群根据自动故障机制，将从节点提升为主节点。这时集群恢复可用。

## Redis Cluster的优势和不足

### 优势

1. 无中心架构。
2. 数据按照slot存储分布在多个节点，节点间数据共享，可动态调整数据分布。
3. 可扩展性，可线性扩展到1000个节点，节点可动态添加或删除。
4. 高可用性，部分节点不可用时，集群仍可用。通过增加 Slave 做 standby 数据副本，能够实现故障自动failover，节点之间通过 gossip 协议交换状态信息，用投票机制完成 Slave 到 Master 的角色提升。
5. 降低运维成本，提高系统的扩展性和可用性。

### 不足

1. Client实现复杂，驱动要求实现Smart Client，缓存slots mapping信息并及时更新，提高了开发难度，客户端的不成熟影响业务的稳定性。目前仅JedisCluster相对成熟，异常处理部分还不完善，比如常见的“max redirect exception”。
2. 节点会因为某些原因发生阻塞（阻塞时间大于clutser-node-timeout），被判断下线，这种 failover 是没有必要的。
3. 数据通过异步复制,不保证数据的强一致性。
4. 多个业务使用同一套集群时，无法根据统计区分冷热数据，资源隔离性较差，容易出现相互影响的情况。
5. Slave在集群中充当“冷备”，不能缓解读压力，当然可以通过SDK的合理设计来提高Slave资源的利用率。

## Redis Cluster在业界有哪些探索

通过调研了解，目前业界使用Redis Cluster大致可以总结为4类：

### 直连型

直连型，又可以称之为经典型或者传统型，是官方的默认使用方式，架构图见图6。这种使用方式的优缺点在上面的介绍中已经有所说明，这里不再过多重复赘述。但值得一提的是，这种方式使用Redis Cluster需要依赖Smart Client，诸如连接维护、缓存路由表、MultiOp和Pipeline的支持都需要在Client上实现，而且很多语言的Client目前都还是没有的（关于Clients的更多介绍请参考<https://redis.io/clients>）。虽然Client能够进行定制化，但有一定的开发难度，客户端的不成熟将直接影响到线上业务的稳定性。

![图6  Redis Cluster架构（来自互联网）](E:\markdown笔记\图片\58db232c7613c.jpg)



图6 Redis Cluster架构



### 带Proxy型

在Redis Cluster还没有那么稳定的时候，很多公司都已经开始探索分布式Redis的实现了，比如有基于Twemproxy或者Codis的实现，下面举一个唯品会基于Twemproxy架构的例子（不少公司分布式Redis的集群架构都经历过这个阶段），如图7所示。

![图7  Redis基于Twemproxy的架构实现（来自互联网）](E:\markdown笔记\图片\58db25961bad6.jpg)



图7 Redis基于Twemproxy的架构实现



这种架构的优点和缺点也比较明显。

**优点：**

1. 后端Sharding逻辑对业务透明，业务方的读写方式和操作单个Redis一致；
2. 可以作为Cache和Storage的Proxy，Proxy的逻辑和Redis资源层的逻辑是隔离的；
3. Proxy层可以用来兼容那些目前还不支持的Clients。

**缺点：**

1. 结构复杂，运维成本高；
2. 可扩展性差，进行扩缩容都需要手动干预；
3. failover逻辑需要自己实现，其本身不能支持故障的自动转移；
4. Proxy层多了一次转发，性能有所损耗。

正是因此，我们知道Redis Cluster和基于Twemproxy结构使用中各自的优缺点，于是就出现了下面的这种架构，糅合了二者的优点，尽量规避二者的缺点，架构如图8。

![图8  Smart Proxy方案架构](E:\markdown笔记\图片\58db25fa11d13.jpg)



图8 Smart Proxy方案架构



目前业界Smart Proxy的方案了解到的有基于Nginx Proxy和自研的，自研的如饿了么开源部分功能的Corvus，优酷土豆是则通过Nginx来实现，滴滴也在展开基于这种方式的探索。选用Nginx Proxy主要是考虑到Nginx的高性能，包括异步非阻塞处理方式、高效的内存管理、和Redis一样都是基于epoll事件驱动模式等优点。优酷土豆的Redis服务化就是采用这种结构。

**优点：**

1. 提供一套HTTP Restful接口，隔离底层资源，对客户端完全透明，跨语言调用变得简单；
2. 升级维护较为容易，维护Redis Cluster，只需平滑升级Proxy；
3. 层次化存储，底层存储做冷热异构存储；
4. 权限控制，Proxy可以通过密钥管理白名单，把一些不合法的请求都过滤掉，并且也可以对用户请求的超大value进行控制和过滤；
5. 安全性，可以屏蔽掉一些危险命令，比如keys *、save、flushall等，当然这些也可以在Redis上进行设置；
6. 资源逻辑隔离，根据不同用户的key加上前缀，来实现动态路由和资源隔离；
7. 监控埋点，对于不同的接口进行埋点监控。

**缺点：**

1. Proxy层做了一次转发，性能有所损耗；
2. 增加了运维成本和管理成本，需要对架构和Nginx Proxy的实现细节足够了解，因为Nginx Proxy在批量接口调用高并发下可能会瞬间向Redis Cluster发起几百甚至上千的协程去访问，导致Redis的连接数或系统负载的不稳定，进而影响集群整体的稳定性。

### 云服务型

这种类型典型的案例就是企业级的PaaS产品，如亚马逊和阿里云提供的Redis Cluster服务，用户无需知道内部的实现细节，只管使用即可，降低了运维和开发成本。当然也有开源的产品，国内如搜狐的CacheCloud，它提供一个Redis云管理平台，实现多种类型（Redis Standalone、Redis Sentinel、Redis Cluster）自动部署，解决Redis实例碎片化现象，提供完善统计、监控、运维功能，减少开发人员的运维成本和误操作，提高机器的利用率，提供灵活的伸缩性，提供方便的接入客户端，更多细节请参考：[https://cachecloud.github.io](https://cachecloud.github.io/)。尽管这还不错，如果是一个新业务，到可以尝试一下，但若对于一个稳定的业务而言，要迁移到CacheCloud上则需要谨慎。如果对分布式框架感兴趣的可以看下Twitter开源的一个实现Memcached和Redis的分布式缓存框架Pelikan，目前国内并没有看到这样的应用案例，它的官网是<http://twitter.github.io/pelikan/>。

![图9  CacheCloud平台架构（来自互联网）](E:\markdown笔记\图片\58db26f58811d.jpg)



图9 CacheCloud平台架构



### 自研型

这种类型在众多类型中更显得孤独，因为这种类型的方案更多是现象级，仅仅存在于为数不多的具有自研能力的公司中，或者说这种方案都是各公司根据自己的业务模型来进行定制化的。这类产品的一个共同特点是没有使用Redis Cluster的全部功能，只是借鉴了Redis Cluster的某些核心功能，比如说failover和slot的迁移。作为国内使用Redis较早的公司之一，新浪微博就基于内部定制化的Redis版本研发出了微博Redis服务化系统Tribe。它支持动态路由、读写分离（从节点能够处理读请求）、负载均衡、配置更新、数据聚集（相同前缀的数据落到同一个slot中）、动态扩缩容，以及数据落地存储。同类型的还有百度的BDRP系统。

![图10  Tribe系统架构图](E:\markdown笔记\图片\58db27163be1b.jpg)



图10 Tribe系统架构图



## Redis Cluster运维开发最佳实践经验

- 根据公司的业务模型选择合适的架构，适合自己的才是最好的；
- 做好容错机制，当连接或者请求异常时进行连接retry或reconnect；
- 重试时间可设置大于cluster-node-time (默认15s)，增强容错性，减少不必要的failover；
- 避免产生hot-key，导致节点成为系统的短板；
- 避免产生big-key，导致网卡打爆和慢查询；
- 设置合理的TTL，释放内存。避免大量key在同一时间段过期，虽然Redis已经做了很多优化，仍然会导致请求变慢；
- 避免使用阻塞操作（如save、flushall、flushdb、keys *等），不建议使用事务；
- Redis Cluster不建议使用pipeline和multi-keys操作（如mset/mget. multi-key操作），减少max redirect的产生；
- 当数据量很大时，由于复制积压缓冲区大小的限制，主从节点做一次全量复制导致网络流量暴增，建议单实例容量不要分配过大或者借鉴微博的优化采用增量复制的方式来规避；
- 数据持久化建议在业务低峰期操作，关闭aofrewrite机制，aof的写入操作放到bio线程中完成，解决磁盘压力较大时Redis阻塞的问题。设置系统参数vm.overcommit_memory=1，也可以避免bgsave/aofrewrite的失败；
- client buffer参数调整 
  client-output-buffer-limit normal 256mb 128mb 60 
  client-output-buffer-limit slave 512mb 256mb 180
- 对于版本升级的问题，修改源码，将Redis的核心处理逻辑封装到动态库，内存中的数据保存在全局变量里，通过外部程序来调用动态库里的相应函数来读写数据。版本升级时只需要替换成新的动态库文件即可，无须重新载入数据，可毫秒级完成；
- 对于实现异地多活或实现数据中心级灾备的要求（即实现集群间数据的实时同步），可以参考搜狐的实现：Redis Cluster => Redis-Port => Smart proxy => Redis Cluster；
- 从Redis 4.2的Roadmap来看，更值得期待（详情：<https://gist.github.com/antirez/a3787d538eec3db381a41654e214b31d>）：
  - 加速key->hashslot的分配
  - 更好更多的数据中心存储
  - redis-trib的C代码将移植到redis-cli，瘦身包体积
  - 集群的备份/恢复
  - 非阻塞的Migrate
  - 更快的resharding
  - 隐藏一个只Cache模式，当没有Slave时，Masters当在有一个失败后能够自动重新分配slot
  - Cluster API和Redis Modules的改进，并且Disque分布式消息队列将作为Redis Module加入Redis。





# 一、Redis主从复制

主从复制：主节点负责写数据，从节点负责读数据，主节点定期把数据同步到从节点保证数据的一致性

### 1. 主从复制的相关操作

**a,配置主从复制方式一、新增redis6380.conf, 加入 slaveof 192.168.152.128 6379, 在6379启动完后再启6380，完成配置；**
b,配置主从复制方式二、redis-server --slaveof 192.168.152.128 6379 临时生效

c,查看状态：info replication
d,断开主从复制：在slave节点，执行6380:>slaveof no one
e,断开后再变成主从复制：6380:> slaveof 192.168.152.128 6379
f,数据较重要的节点，主从复制时使用密码验证： requirepass
e,**从节点建议用只读模式slave-read-only=yes, 若从节点修改数据，主从数据不一致** 
h,传输延迟：主从一般部署在不同机器上，复制时存在网络延时问题，redis提供repl-disable-tcp-nodelay参数决定是否关闭TCP_NODELAY,默认为关闭
参数关闭时：无论大小都会及时发布到从节点，占带宽，适用于主从网络好的场景，
参数启用时：主节点合并所有数据成TCP包节省带宽，默认为40毫秒发一次，取决于内核，主从的同步延迟40毫秒，适用于网络环境复杂或带宽紧张，如跨机房

### 2. Redis主从拓扑

a)一主一从：用于主节点故障转移从节点，当主节点的“写”命令并发高且需要持久化，可以只在从节点开启AOF（主节点不需要），这样即保证了数据的安全性，也避免持久化对主节点的影响 

 ![img](E:\markdown笔记\图片\1227483-20180201102310015-486760227.png)

b)一主多从：针对“读”较多的场景，“读”由多个从节点来分担，但节点越多，主节点同步到多节点的次数也越多，影响带宽，也加重主节点的稳定

 ![img](E:\markdown笔记\图片\1227483-20180201103217750-831662244.png)

c)树状主从：一主多从的缺点（主节点推送次数多压力大）可用些方案解决，主节点只推送一次数据到从节点B，再由从节点B推送到C，减轻主节点推送的压力。

 

![img](E:\markdown笔记\图片\1227483-20180201103511703-1604168118.png)

### 3. 主从复制原理

 ![img](E:\markdown笔记\图片\1227483-20180201103759312-1678395133.png)

### 4. 数据同步

redis 2.8 版本以上使用psync命令完成同步，过程分“全量”与“部分”复制
**全量复制：**一般用于初次复制场景（第一次建立SLAVE后全量）
**部分复制：**网络出现问题，从节点再次连接主节点时，主节点补发缺少的数据，每次数据增量同步
**心跳：**主从有长连接心跳，主节点默认每10S向从节点发ping命令，repl-ping-slave-period控制发送频率

### 5. 主从的缺点

a)主从复制，若主节点出现问题，则不能提供服务，需要人工修改配置将从变主
b)主从复制主节点的写能力单机，能力有限
c)单机节点的存储能力也有限

### 6.主从故障如何故障转移

a)主节点(master)故障，从节点slave-1端执行 slaveof no one后变成新主节点；
b)其它的节点成为新主节点的从节点，并从新节点复制数据；
c)需要人工干预，无法实现高可用。

![img](E:\markdown笔记\图片\1227483-20180201105443187-409161127.png)

# 二、Redis哨兵机制（Sentinel）

**\1. 为什么要有哨兵机制？**

​       哨兵机制的出现是为了解决主从复制的缺点的

**\2. 哨兵机制(sentinel)的高可用**

　　原理：当主节点出现故障时，由Redis Sentinel自动完成故障发现和转移，并通知应用方，实现高可用性。

![img](E:\markdown笔记\图片\1227483-20180201110330296-1920426355.png)

其实整个过程只需要一个哨兵节点来完成，首先使用Raft算法（选举算法）实现选举机制，选出一个哨兵节点来完成转移和通知

### 3. 哨兵的定时监控任务

**任务1：**每个哨兵节点每10秒会向主节点和从节点发送info命令获取最拓扑结构图，哨兵配置时只要配置对主节点的监控即可，通过向主节点发送info，获取从节点的信息，并当有新的从节点加入时可以马上感知到

![img](E:\markdown笔记\图片\1227483-20180201113518031-1426392070.png)

**任务2：**每个哨兵节点每隔2秒会向redis数据节点的指定频道上发送该哨兵节点对于主节点的判断以及当前哨兵节点的信息，同时每个哨兵节点也会订阅该频道，来了解其它哨兵节点的信息及对主节点的判断，其实就是通过消息publish和subscribe来完成的

![img](E:\markdown笔记\图片\1227483-20180201113623921-1930278582.png)

 **任务3：**每隔1秒每个哨兵会向主节点、从节点及其余哨兵节点发送一次ping命令做一次心跳检测，这个也是哨兵用来判断节点是否正常的重要依据

![img](E:\markdown笔记\图片\1227483-20180201115230609-828804435.png)

客观下线：当主观下线的节点是主节点时，此时该哨兵3节点会通过指令sentinel is-masterdown-by-addr寻求其它哨兵节点对主节点的判断，当超过quorum（选举）个数，此时哨兵节点则认为该主节点确实有问题，这样就客观下线了，大部分哨兵节点都同意下线操作，也就说是客观下线

 ![img](E:\markdown笔记\图片\1227483-20180201115626875-1938202974.png)

### 4. 领导者哨兵选举流程

a)每个在线的哨兵节点都可以成为领导者，当它确认（比如哨兵3）主节点下线时，会向其它哨兵发is-master-down-by-addr命令，征求判断并要求将自己设置为领导者，由领导者处理故障转移；
b)当其它哨兵收到此命令时，可以同意或者拒绝它成为领导者；
c)如果哨兵3发现自己在选举的票数大于等于num(sentinels)/2+1时，将成为领导者，如果没有超过，继续选举…………

 ![img](E:\markdown笔记\图片\1227483-20180201121410109-4834991.png)

### 5. 故障转移机制

a)由Sentinel节点定期监控发现主节点是否出现了故障

sentinel会向master发送心跳PING来确认master是否存活，如果master在“一定时间范围”内不回应PONG 或者是回复了一个错误消息，那么这个sentinel会主观地(单方面地)认为这个master已经不可用了

 ![img](E:\markdown笔记\图片\1227483-20180201121656031-220411144.png)

 

 b) 当主节点出现故障，此时3个Sentinel节点共同选举了Sentinel3节点为领导，负载处理主节点的故障转移

![img](E:\markdown笔记\图片\1227483-20180201121820875-869825604.png)

 

 c) 由Sentinel3领导者节点执行故障转移，过程和主从复制一样，但是自动执行

 ![img](E:\markdown笔记\图片\1227483-20180201121911671-2038984859.png)

 流程：

　　  1. 将slave-1脱离原从节点，升级主节点，

​         \2. 将从节点slave-2指向新的主节点

​         \3. 通知客户端主节点已更换

​         \4. 将原主节点（oldMaster）变成从节点，指向新的主节点

 d) 故障转移后的redis sentinel的拓扑结构图

![img](E:\markdown笔记\图片\1227483-20180201122103890-1930925517.png)

### 6. 哨兵机制－故障转移详细流程-确认主节点

a) 过滤掉不健康的（下线或断线），没有回复过哨兵ping响应的从节点

b) 选择salve-priority从节点优先级最高（redis.conf）的

c) 选择复制偏移量最大，指复制最完整的从节点

### **7. 实战：如何安装和部署哨兵**

以3个Sentinel节点、2个从节点、1个主节点为例进行安装部署

![img](E:\markdown笔记\图片\1227483-20180204102431967-1066194410.png)

**1. 前提：**先搭好一主两从redis的主从复制，和之前的主从复制搭建一样，搭建方式如下：

　　A）主节点6379节点（/usr/local/bin/conf/redis6379.conf）：

　　　　修改 requirepass 12345678，注释掉#bind 127.0.0.1

　　B) 从节点redis6380.conf和redis6381.conf: 配置都一样

 

　　　　修改 requirepass 12345678 ,注释掉#bind 127.0.0.1,

　　　　加上访问主节点的密码masterauth 12345678 ,加上slaveof 192.168.152.128 6379

​    **注意****：**当主从起来后，主节点可读写，从节点只可读不可写

**2. redis sentinel哨兵机制核心配置**(也是3个节点)：

​       /usr/local/bin/conf/sentinel_26379.conf  

​       /usr/local/bin/conf/sentinel_26380.conf

​       /usr/local/bin/conf/sentinel_26381.conf

将三个文件的端口改成: 26379   26380   26381

然后：sentinel monitor mymaster 192.168.152.128 **6379** 2  //监听主节点6379

​      sentinel auth-pass mymaster 12345678     //连接主节点时的密码

**三个配置除端口外，其它一样。**

**3. 哨兵其它的配置**：只要修改每个sentinel.conf的这段配置即可：

sentinel monitor mymaster 192.168.152.128 6379 2  

//监控主节点的IP地址端口，sentinel监控的master的名字叫做mymaster，2代表，当集群中有2个sentinel认为master死了时，才能真正认为该master已经不可用了

sentinel auth-pass mymaster 12345678  //sentinel连主节点的密码

sentinel config-epoch mymaster 2  //故障转移时最多可以有2从节点同时对新主节点进行数据同步

sentinel leader-epoch mymaster 2

sentinel failover-timeout mymasterA **180000** //故障转移超时时间180s，                            

a,如果转移超时失败，下次转移时时间为之前的2倍；

b,从节点变主节点时，从节点执行slaveof no one命令一直失败的话，当时间超过**180S**时，则故障转移失败

c,从节点复制新主节点时间超过**180S**转移失败

sentinel down-after-milliseconds mymasterA **300000**//sentinel节点定期向主节点ping命令，当超过了**300S**时间后没有回复，可能就认定为此主节点出现故障了……

sentinel parallel-syncs mymasterA **1** //故障转移后，**1**代表每个从节点按顺序排队一个一个复制主节点数据，如果为3，指3个从节点同时并发复制主节点数据，不会影响阻塞，但存在网络和IO开销

**4. 启动redis服务和sentinel服务:**

**a)先把之前安装的redis里面的标绿色的文件都拷贝到 usr/local/bin目录下，然后再再bin目录下新建一个conf文件夹存放配置好的redis主从配置文件和哨兵配置文件**

**![img](E:\markdown笔记\图片\1227483-20180204115936420-1099845747.png)**

![img](E:\markdown笔记\图片\1227483-20180204120442185-1065379484.png)

b)启动主从复制服务，先启动主再启动从

主：./redis-server conf/redis6379.conf &

从：

　　./redis-server conf/redis6380.conf &

　　./redis-server conf/redis6381.conf &      

![img](E:\markdown笔记\图片\1227483-20180204120947029-1058769480.png)

![img](E:\markdown笔记\图片\1227483-20180204121142107-1214057093.png)

 

**c）启动sentinel服务:**

​       ./redis-sentinel conf/sentinel_26379.conf &

![img](E:\markdown笔记\图片\1227483-20180204121232404-1080606138.png)

​       ./redis-sentinel conf/sentinel_26380.conf &

![img](E:\markdown笔记\图片\1227483-20180204121333139-346570284.png)

​     

 ./redis-sentinel conf/sentinel_26381.conf &

![img](E:\markdown笔记\图片\1227483-20180204121425607-1422147749.png)

到此服务全部启动完毕

![img](E:\markdown笔记\图片\1227483-20180204121531139-1303538630.png)

 

连接到6379的redis的服务，可看到6379就是主节点，他有6380和6381两个从节点

**![img](E:\markdown笔记\图片\1227483-20180204121924529-462773287.png)**

**5. 测试：**kill -9 6379  杀掉6379的redis服务

可以看到杀掉6379以后6380变为了主节点，6381变为了6380的从节点

![img](E:\markdown笔记\图片\1227483-20180204122537264-61572248.png)

![img](E:\markdown笔记\图片\1227483-20180204122711014-213045075.png)

![img](E:\markdown笔记\图片\1227483-20180204122755904-130617083.png)

重新启动6379以后变为6380的从节点

![img](E:\markdown笔记\图片\1227483-20180204122959045-163078668.png)

![img](E:\markdown笔记\图片\1227483-20180204123043264-711236759.png)

看日志是分配6380 是6381的主节点，当6379服务再启动时，已变成从节点

假设6380升级为主节点:进入6380>info replication     可以看到role:master

打开sentinel_26379.conf等三个配置，sentinel monitor mymaster 192.168.152.128 **6380** 2

打开redis6379.conf等三个配置, slaveof 192.168.152.128 6380,也变成了6380

**注意：生产环境建议让redis Sentinel部署到不同的物理机上。**

### 8.部署建议

a，sentinel节点应部署在多台物理机（线上环境）

b，至少三个且奇数个sentinel节点

c，通过以上我们知道，3个sentinel可同时监控一个主节点或多个主节点

​    监听N个主节点较多时，如果sentinel出现异常，会对多个主节点有影响，同时还会造成sentinel节点产生过多的网络连接，

​    一般线上建议还是， 3个sentinel监听一个主节点





# Redis哨兵的详解

## 1 哨兵的作用

哨兵是redis集群架构中非常重要的一个组件，主要功能如下： 

1. 集群监控：负责监控redis master和slave进程是否正常工作 
2. 消息通知：如果某个redis实例有故障，那么哨兵负责发送消息作为报警通知给管理员 
3. 故障转移：如果master node挂掉了，会自动转移到slave node上 
4. 配置中心：如果故障转移发生了，通知client客户端新的master地址

## 2 哨兵的核心知识

1. 故障转移时，判断一个master node是宕机了，需要大部分的哨兵都同意才行，涉及到了分布式选举的问题
2. 哨兵至少需要3个实例，来保证自己的健壮性
3. 哨兵 + redis主从的部署架构，是不会保证数据零丢失的，只能保证redis集群的高可用性

## 3 sdown和odown

1. sdown和odown两种失败状态
2. sdown是主观宕机，就一个哨兵如果自己觉得一个master宕机了，那么就是主观宕机
3. odown是客观宕机，如果quorum数量的哨兵都觉得一个master宕机了，那么就是客观宕机
4. sdown达成的条件：如果一个哨兵ping一个master，超过了is-master-down-after-milliseconds指定的毫秒数之后，就主观认为master宕机
5. odown达成的条件：如果一个哨兵在指定时间内，收到了quorum指定数量的其他哨兵也认为那个master是sdown了，那么就认为是odown了，客观认为master宕机

## 4 quorum和majority

1. quorum：确认odown的最少的哨兵数量
2. majority：授权进行主从切换的最少的哨兵数量
3. 每次一个哨兵要做主备切换，首先需要quorum数量的哨兵认为odown，然后选举出一个哨兵来做切换，这个哨兵还得得到majority哨兵的授权，才能正式执行切换
4. 如果quorum < majority，比如5个哨兵，majority就是3，quorum设置为2，那么就3个哨兵授权就可以执行切换，但是如果quorum >= majority，那么必须quorum数量的哨兵都授权，比如5个哨兵，quorum是5，那么必须5个哨兵都同意授权，才能执行切换

## 5 为什么哨兵至少3个节点

哨兵集群必须部署2个以上节点。如果哨兵集群仅仅部署了个2个哨兵实例，那么它的majority就是2（2的majority=2，3的majority=2，5的majority=3，4的majority=2），如果其中一个哨兵宕机了，就无法满足majority>=2这个条件，那么在master发生故障的时候也就无法进行主从切换。

## 6 脑裂以及redis数据丢失

主备切换的过程，可能会导致数据丢失 
（1）异步复制导致的数据丢失 
因为master -> slave的复制是异步的，所以可能有部分数据还没复制到slave，master就宕机了，此时这些部分数据就丢失了 
（2）脑裂导致的数据丢失 
脑裂，也就是说，某个master所在机器突然脱离了正常的网络，跟其他slave机器不能连接，但是实际上master还运行着 
此时哨兵可能就会认为master宕机了，然后开启选举，将其他slave切换成了master，这个时候，集群里就会有两个master，也就是所谓的脑裂。 
此时虽然某个slave被切换成了master，但是可能client还没来得及切换到新的master，还继续写向旧master的数据可能也丢失了，因此旧master再次恢复的时候，会被作为一个slave挂到新的master上去，自己的数据会清空，重新从新的master复制数据

## 7 如何尽可能减少数据丢失

下面两个配置可以减少异步复制和脑裂导致的数据丢失：

```
`min-slaves-to-write ``1``min-slaves-max-lag ``10`
```



解释：要求至少有1个slave，数据复制和同步的延迟不能超过10秒，如果说一旦所有的slave，数据复制和同步的延迟都超过了10秒钟，那么这个时候，master就不会再接收任何请求了 
（1）减少异步复制的数据丢失 
有了min-slaves-max-lag这个配置，就可以确保说，一旦slave复制数据和ack延时太长，就认为可能master宕机后损失的数据太多了，那么就拒绝写请求，这样可以把master宕机时由于部分数据未同步到slave导致的数据丢失降低的可控范围内 
（2）减少脑裂的数据丢失 
如果一个master出现了脑裂，跟其他slave丢了连接，那么上面两个配置可以确保说，如果不能继续给指定数量的slave发送数据，而且slave超过10秒没有给自己ack消息，那么就直接拒绝客户端的写请求，这样脑裂后的旧master就不会接受client的新数据，也就避免了数据丢失 
上面的配置就确保了，如果跟任何一个slave丢了连接，在10秒后发现没有slave给自己ack，那么就拒绝新的写请求。
因此在脑裂场景下，最多就丢失10秒的数据

## 8 哨兵集群的自动发现机制

哨兵互相之间的发现，是通过redis的pub/sub系统实现的，每个哨兵都会往sentinel:hello这个channel里发送一个消息，这时候所有其他哨兵都可以消费到这个消息，并感知到其他的哨兵的存在 
每隔两秒钟，每个哨兵都会往自己监控的某个master+slaves对应的sentinel:hello channel里发送一个消息，内容是自己的host、ip和runid还有对这个master的监控配置 
每个哨兵也会去监听自己监控的每个master+slaves对应的sentinel:hello channel，然后去感知到同样在监听这个master+slaves的其他哨兵的存在 
每个哨兵还会跟其他哨兵交换对master的监控配置，互相进行监控配置的同步

## 9 slave配置的自动纠正

哨兵会负责自动纠正slave的一些配置，比如slave如果要成为潜在的master候选人，哨兵会确保slave在复制现有master的数据; 如果slave连接到了一个错误的master上，比如故障转移之后，那么哨兵会确保它们连接到正确的master上

## 10 master选举算法

如果一个master被认为odown了，而且majority哨兵都允许了主备切换，那么某个哨兵就会执行主备切换操作，此时首先要选举一个slave来。 
选举的时候会考虑slave的一些信息： 

1. 跟master断开连接的时长 
2. slave优先级 
3. 复制offset 
4. run id 

如果一个slave跟master断开连接已经超过了down-after-milliseconds的10倍，外加master宕机的时长，那么slave就被认为不适合选举为master，计算公式如下：

```
`(down-after-milliseconds * ``10``) + milliseconds_since_master_is_in_SDOWN_state`
```

　　

接下来会对slave进行排序 
（1）按照slave优先级进行排序，slave priority越低，优先级就越高 
（2）如果slave priority相同，那么看replica offset，哪个slave复制了越多的数据，offset越靠后，优先级就越高 
（3）如果上面两个条件都相同，那么选择一个run id比较小的那个slave

## 11 configuration epoch

哨兵会对一套redis master+slave进行监控，有相应的监控的配置 
执行切换的那个哨兵，会从要切换到的新master（salve->master）那里得到一个configuration epoch，这就是一个version号，每次切换的version号都必须是唯一的 
如果第一个选举出的哨兵切换失败了，那么其他哨兵，会等待failover-timeout时间，然后接替继续执行切换，此时会重新获取一个新的configuration epoch，作为新的version号

## 12 configuraiton传播

哨兵完成切换之后，会在自己本地更新生成最新的master配置，然后同步给其他的哨兵，就是通过之前说的pub/sub消息机制 
这里之前的version号就很重要了，因为各种消息都是通过一个channel去发布和监听的，所以一个哨兵完成一次新的切换之后，新的master配置是跟着新的version号的 
其他的哨兵都是根据版本号的大小来更新自己的master配置的