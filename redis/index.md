[TOC]

[中文官网](http://www.redis.cn/documentation.html)
# 数据结构
- 简单动态字符串 SDS
    - 读取字符串长度时间复杂度: o(1)
    - 相比c语言的字符串：
        -  减少读取时间复杂度
        -  减少修改字符串时内存重分配（如果字符串减少SDS不需要重新分配）
        -  可以直接存放二进制数据。
    - 内部结构如下

```c
{
    // 已使用的字节数组长度
    int len;
    // 未使用的字节数组长度
    int free;
    // 存放数据的字节数组，其最后以为存放‘\0’
    // 因此buf.length  = len + free + 1;
    char[] buf;
}
```

- 链表 linkedlist
    - 获取链表长度时间负责读：o(1) 
    - 优势：
        - 可以指向前后节点，支持**从头/从尾**插入数据，时间复杂度是o(1)
        - 不会造成环，头部/尾部节点都指向null
        - 获取链表长度的时间复杂度是o(1)
        - 节点存放的是指向值的指针，即**支持不同类型数据**
    - 内部结构如下

```c
{
    // 指向前一个节点（头节点指向nil）
    listNode pre;
    // 指向后一个节点（尾节点指向nil）
    listNode next;
    // 当前节点值
    void value;
}listNode
{
    // 头节点
    listNode.head;
    // 尾节点
    listNode.tail;
    // 链表节点数
    long len
    // 节点值复制函数
    // 节点值释放函数
    // 节点值比较函数
}list
```

- 字典 
    - 哈希表底层实现
    - 其与hashTable实现方式类型，用数组 + 链表的方式
    - 内部结构如下

```c
{
    // 哈希表数组
    dictEntry table
    // 哈希表大小
    long size
    // 哈希表大小掩码
    // 与key的hash值计算数组下标索引
    long sizemask
    // 已使用节点数
    long used
}dictht
{
    // 健
    void key;
    // 值
    union{
        void val;
        uint64_tu64;
        int64_ts64;
    }v;
    // 指向下一个节点，形成链表
    dicEntry next
}dictEntry
```

- 跳跃表 skiplist
    - 时间复杂度o(logN) 
    - 空间换取时间的数据结构
        - 第一层是**双向链表**(方便倒叙获取数据)存储所有数据
        - 高层为单向列表，当节点插入时随机生成层数（max=64）
            - 插入新层，头节点->它，它->尾节点
            - 插入已有层数，同层的前一节点->它，它->尾节点
        - 因此层数越高，数据量越大，跳跃性也越大，查询时从最高层开始
    - zset底层是字典 + 跳跃表实现，分数排序通过字典实现然后再跳表中查询对应值，跳表不支持范围查询
    - 对比树结构，跳表节省内存、对于插入/删除操作时间复杂度底、实现方式简单
    - 内部结构如下：

```c
{
    // 层
    {
        // 当前层下一个节点
        zskiplistNode forward;
        // 跨度
        int span;
    }level[];
    
    // 后退节点
    zskiplistNode backward;
    // 分值
    double score;
    // 成员对象
    robj obj;
}zskiplistNode
{
    // 头节点
    skiplistNode header;
    // 尾节点
    skiplistNode tail;
    // 节点数量
    long length;
    // 最大层数
    int leve
}zskiplist
```

- 整数集合 intset
    - 简单整型set数据结构 
    - 内部结构如下

```
{
    // 编码方式
    encoding;
    // 元素长度
    length;
    // 保存元素数组
    contents[];
}intset
```

- 压缩列表 ziplist
    - 当列表含少量列表项（小整型/短字符串）时十一压缩列表
    - 内部结构如下

```
{
    // 前一个节点字节长度
    previous_entry_length;
    // 编码方式
    encoding;
    // 内容
    content;
}
```

- 对象 redisObject
    - type: 对象类型
    - endocing: 对象内部编码
    - lru: 最后一次被命令访问时间
    - refcount: 被引用次数
    - ptr: 具体数据

类型 | 编码 | 对象
---|---|---
String | int | 整数实现的字符串对象
String | embstr | 基于embstr编码实现的sds字符串对象
String | raw | sds字符串对
List | zipList | 压缩列表实现的列表对象
List | linkedlist | 双向链表实现的列表对象
hash | linkedlist | 压缩列表实现的哈希对象
hash | linkedlist | 字典实现的哈希对象
set | linkedlist | 整数集合实现的集合对象
set | linkedlist | 字典实现的集合对象
zset | linkedlist | 压缩列表实现的有序集合
zset | linkedlist | 跳表+字典实现的有序集合

# 缓存淘汰策略
- volatile-lru: 已设置过期时间数据集中挑选最近最少使用数据淘汰
- volatile-ttl: 已设置过期时间数据集中挑选将要过期的数据淘汰
- volatile-random: 已设置过期时间数据集中随机淘汰
- allkeys-lru: 数据集中挑选最近最少使用数据淘汰
- allkeys-random: 数据集中随机淘汰
- no-enviction: 禁止淘汰（不用）


# 内存分配
> 查看reids内存分配     

```shell
info memory
// 结果
// 分配内存总量（含虚拟内存）
used_memory:1051040
// 占有操作系统内存总量（不含虚拟内存）
used_memory_rss:2715648
// 内存碎片比例，小于0时使用了虚拟内存
mem_fragmentation_ratio:2.70
// 内存分配器
mem_allocator:libc
```

- 数据
- 进程
- 缓存内存
- 内存碎片

# 持久化
## 持久化方式
- RDB方式 （系统默认）
- AOF方式 

## RDB持久化
> 核心函数RDBSave(生产RDB文件)和RDBLoad(从文件加载到内存)。   
> 按照一定时间周期策略把内存数据以快照形式保存到硬盘掉二进制文件。【时间周期触发前的数据可能丢失】

### 触发时机
1. 符合快照规则
2. 执行save/bgsave命令
3. 支持flushall命令
4. 执行主从复置操作

### 数据恢复
> redis重启后读取rdb快照文件，将数据从硬盘载入内存  
> 一千万字符串类型键/1GB文件需要20～30秒

### RDB配置
> 配置尽量交错配置
- save <seconds> <changes> ： x秒内n个key更改过即进行快照存储（可配置多个）
    - save 900 1 ： 900s内有1个key进行变更即进行快照（最好配置一个1key更新的，否则可能出现一直不快照的情况）
    - save 300 10 ： 300s内有300个key进行变更即进行快照
- dir ./ ： 快照文件目录
- dbfilename dump.rdb ：默认文件名称

### 快照原理
1. 调用操作系统fork函数，复制一个子进程
2. 父进行继续接受客户端命令（**高可用**），子进程将内存数据写入**临时文件**（生成临时文件是确保快照文件每次都是完整的。可能存在写入过程出现宕机，如果直接写入rdb文件会造成文件不完整）
3. 临时文件写入完成后替换旧文件，完成快照操作

### 使用场景
定时备份数据，RDB文件是经过压缩的二进制文件，占有空间小便于传输。

### 优缺点
- 缺点： redis一旦异常退出，就会丢失最后一次快照后更改的数据。可以通过组合设置快照出发条件减少丢失数据数量。但是如果数据比较重要需要使用AOF持久化。
- 优点： 最大化redis性能。父进行不需要执行磁盘的I/O操作，保存工作完成有子进程完成。但是如果数据集较大时fork比较耗时，造成服务器一段时间停止处理客户端请求（**非高可用**）

## AOF持久化
> 默认redis不开启aof持久化  
> 如果开启，redis优先使用aof文件还原数据
### AOF配置
- appendonly yes ：开启aof持久化
- dir ./ ：默认保存文件地址
- appendfilename appendonly.aof ：默认aof文件名称

### 触发时机
- 每执行一条更改数据命令，redis将该**命令**写入aof文件。
    - 触发频繁降低redis性能（大部分情况可接受）
    - 使用较快的硬盘可提高aof性能
- aof机制将命令记录在aof文件时，由于操作系统的缓存机制，不是实时写入，而是写入硬盘缓存，再通过硬盘缓存机制刷新到文件
- 参数
    - appendfsync always ： 每次执行即写入，性能低安全高
    - appendfsync everysec ： 每秒执行一次
    - appendfsync no ：有操作系统执行，性能高安全地

### aof重写 （优化aof文件）
- redis在aof文件体积过大时自动在后台对aof文件进行重写，重写后文件包含恢复当前数据集所需最小命令集合。
    - aof文件记录写入命令依照redis协议格式，内容容易解析。
    - 过程（安全）： redis在创建新的aof文件过程中，会继续将命令追加到**现有aof文件**，即使aof重写过程宕机文件内容也不会丢失。一旦新的aof文件创建完成，reids将旧aof文件切换到新aof文件。开始对新aof文件进行追加操作。
    - 参数：
        - auto-aof-rewrite-percentage 100 ：文件超过百分比后进行重写
        - auto-aof-rewrite-min-size 64mb ：文件小于64mb不进行优化
- aof损坏修复（？）： 命令写入未完成时停机，重启时拒绝载入aof文件，确保数据一致性。
    - 创建备份aof文件
    - 使用redis-check-aof程序进行修复
        - redis-check-aof --fix readonly.aof
    - 重启redis服务，等待服务器载入修复后的aof文件，进行数据恢复


# 架构模式
## 单机版
> 简单的单机服务

#### 缺点
> 1. 内存容量有限
> 2. 处理能力有限
> 3. 无法高可用

## 主从复制
> 主从复制（replication），slave节点能精确的复制master节点的内容，即使slave节点断开重连时也会同步内容保持是master节点精确副本。

### 运行机制
- 当一个master和一个slave正常连接时，master 会发送一连串的命令流来保持对 slave 的更新，以便于将自身数据集的改变复制给 slave ， ：包括客户端的写入、key 的过期或被逐出等等
- slave 断开重新连接上 master 并会尝试进行部分重同步：这意味着它会尝试只获取在断开连接期间内丢失的命令流
- 当无法进行部分重同步时， slave 会请求进行全量重同步。 master 需要创建所有数据的快照，将之发送给 slave ，之后在数据集更改时持续发送命令流到 slave 

## 特点
- 默认异步复制，低延迟、高性能
- 一个master可以有多个slave
- 多个slave之间可以互连
- 复制时redis是非阻塞的，可以继续处理查询请求
- 为避免master磁盘开销可以设置master节点不出久化，而slave节点同步数据后持久化，但是此方案制需要避免master自动重启，如果master重启后内存数据会被清空，slave再次同步数据时也会清空数据。

### 配置

```
// slave节点设置同步master节点ip:port
slaveif 127.0.0.1:7000

// 设置从节点只读
slave-read-only true

// 解决脑裂/异步复制导致延迟数据丢失（全量复制）问题
// 至少要有3个slave节点与master保持10秒钟以内的数据同步，否则master就不会接受新的请求 (主节点设置)
min-slaves-to-write 3
min-slaves-max-lag 10
```


### 同步原理
##### 类型
- 全量同步（第一次连接主机、重连请求增量同步失败）
- 增量同步

##### 工作方式
> 每个master都有一个replication ID 和偏移量offset，slave节点会根据offset判断需要增量同步那些内容。

##### 全量同步
1. 同步快照阶段：master创建并发送快照到Slave,Slave载入并解析快照，同时master将此阶段产生的新写命令存储到缓冲区。
2. 同步写缓冲阶段：master向Slave同步存储在缓冲区的写操作命令。（此步骤如果master宕机可能导致缓冲区数据丢失）
3. 同步增量阶段：master向Slave同步写操作命令。


##### 增量同步
1. master每执行一个写命令会向slave发送相同写命令，slave接受并执行。


## 哨兵机制
> 解决master动态选举（主从复制不支持）

#### 作用
- 监控：sentinel 会不断检查master和slave是否正常运转。
- 提醒：监控到某个节点出现问题，sentinal通过API向管理员或其他应用发送通知。
- 自动故障转移： 当master不能正常工作时，sentinel开始一次自动古装转移操作
    - 从失效的master的slave中升级新的master，并让失效的matser的其他slave改为复制新的master
    - 客户端试图连接失效master时，会返回新的master地址。
    - master和slave服务切换后，对应的配置文件内容会相应的改变。

#### 哨兵配置

> sentinel.conf 

- sentinel monitor mymaster 127.0.0.1 6379 2
    - 监视一个名为 mymaster 的主服务器， 这个主服务器的 IP 地址为 127.0.0.1 ， 端口号为 6379 ， 而将这个主服务器判断为失效至少需要 2 个 Sentinel 同意
    - 无论你设置要多少个 Sentinel 同意才能判断一个服务器失效， 一个 Sentinel 都需要获得系统中多数（majority） Sentinel 的支持，才能发起一次自动故障迁移（只有少数Sentinel正常运行时不能执行自动故障转移）。
- sentinel down-after-milliseconds mymaster 3000
    - 指定了 Sentinel 认为服务器已经断线所需的毫秒数。
    - 主观下线（SDOWN）：单个 Sentinel 实例对服务器做出的下线判断
    - 客观下线（ODOWN）：多个 Sentinel 实例在对同一个服务器做出 SDOWN 判断， 并且通过 SENTINEL is-master-down-by-addr 命令互相交流之后， 得出的服务器下线判断
- sentinel parallel-syncs mymaster 1 
    - 执行主备切换时多少个slave对master进行同步
    - 从服务器在载入主服务器发来的 RDB 文件时， 仍然会造成从服务器在一段时间内不能处理命令请求，如果全部从服务器一起对新的主服务器进行同步， 那么就可能会造成所有从服务器在短时间内全部不可用的情况出现。
- sentinel failover-timeout mymaster 180000
    - 故障转移开始，三分钟内没有完成，则认为转移失败

#### 启动哨兵

> 动 Sentinel 实例必须指定相应的配置文件， 系统会使用配置文件来保存 Sentinel 的当前状态， 并在 Sentinel 重启时通过载入配置文件来进行状态还原。     
> 如果启动 Sentinel 时没有指定相应的配置文件， 或者指定的配置文件不可写（not writable）， 那么 Sentinel 会拒绝启动

```
redis-server /path/to/sentinel.conf --sentinel
```

#### 故障判定
1. 每个sentinel进程以每秒一次的频率向master主服务、slave从服务以及其他sentinel进程发送一个ping命令。
2. 如果一个实例（master或者slave）最后一次回复ping命令时间时间超过**down-after-milliseconds**值，则该实例被标记为主观下线（sdown）。
3. 如果一个master被标记为主观下线，所有监控master的sentinel进程以每秒一次的频率确认master确实进入主观下线状态
4. 当足够数量（该值在配置文件设置）sentinel进程认为master进入主观下线状态时，标记master为客观下线。
5. 一般情况sentinel向master和slave服务发送info命令频率为10秒一次，但当一个master节点客观下线时，sentinel会以每秒一次的频率向该master节点的所有slave节点发送INFO命令。
6. 如果没有足够数量sentinel同意master主观下线，则取消master客观下线状态。如果master重新回复sentinel发送ping命令，移除主观下线状态。

#### 故障转移
- 发现主服务器已经进入客观下线状态。
- 对我们的当前纪元进行自增， 并尝试在这个纪元中当选(超过半数sentinel 节点同意,并且也大于quorum)。
- 如果当选失败，那么在设定的故障迁移超时时间的两倍之后， 重新尝试当选。 
- 如果当选成功， 那么执行以下步骤。
    - 选出一个从服务器，并将它升级为主服务器。
    - 向被选中的从服务器发送 SLAVEOF NO ONE 命令，让它转变为主服务器。
    - 通过发布与订阅功能， 将更新后的配置传播给所有其他 Sentinel ， 其他 Sentinel 对它们自己的配置进行更新。
    - 向已下线主服务器的从服务器发送 SLAVEOF 命令， 让它们去复制新的主服务器
    - 更新原来master 节点配置为 slave 节点,并保持对其进行关注,一旦这个节点重新恢复正常后,会命令它去复制新的master节点信息

##### 新主服务选择规则
- 在失效主服务器属下的从服务器当中，那些被标记为主观下线、已断线、或者最后一次回复 PING命令的时间大于五秒钟的从服务器都会被淘汰。
- 在失效主服务器属下的从服务器当中， 那些与失效主服务器连接断开的时长超过 down-after 选项指定的时长十倍的从服务器都会被淘汰。
- 在经历了以上两轮淘汰之后剩下来的从服务器中，
    - slave-priority(slave节点优先级)最高的slave节点
    - 复制偏移量（replication offset）最大的服务器
    - runId最小的slave节点(启动最早的节点)

##### 说明
> 纪元所有投票失效服务主观下线的sentinal服务。  
> 执行故障转移只会有一个sentinal执行，由其他纪元中的sentinal选举（配置版本最新）。     
> Sentinel 的状态会被持久化在 Sentinel 配置文件里面

#### sentinel互相发现
> 一个 Sentinel 可以与其他多个 Sentinel 进行连接， 各个 Sentinel 之间可以互相检查对方的可用性， 并进行信息交换。

- 每个 Sentinel 会以每两秒一次的频率， 通过发布与订阅功能， 向被它监视的所有主服务器和从服务器的 sentinel:hello 频道发送一条信息， 信息中包含了 Sentinel 的 IP 地址、端口号和运行 ID （runid）。
- 每个sentinel都订阅自己监听的主/从服务的sentinel:hello 频道，从而发现之前未出现的sentinel，新sentinel会被添加到维护列表中（所有监听该服务的其他sentinel）。
- sentinel 发布的信息中还包括完整的主服务器配置。 如果监听到的Sentinel信息中主服务器配置比自己新则立即升级成新配置。
- Sentinel 会先检查列表中是否已经包含了和要添加的 Sentinel 拥有相同运行 ID 或者相同地址（包括 IP 地址和端口号）的 Sentinel，如果有先移除再添加。

#### sentinel API
> 可以通过命令的方式订阅事件，具体api见http://www.redis.cn/topics/sentinel.html

#### TITL模式（保护机制，确保sentinal可用）
- 触发
    - 两次调用时间之间的差距为负值， 或者非常大（超过 2 秒钟）， 那么 Sentinel 进入 TILT 模式
    - Sentinel 已经进入 TILT 模式， 那么 Sentinel 延迟退出 TILT 模式的时间
- 目标
    - 不再执行任何操作，比如故障转移
    - 当有实例向这个 Sentinel 发送 SENTINEL is-master-down-by-addr 命令时， Sentinel 返回负值： 因为这个 Sentinel 所进行的下线判断已经不再准确
    - TILT 可以正常维持 30 秒钟， 那么 Sentinel 退出 TILT 模式

### 总结
> 3个定时任务
1. 10s一次对master和slave服务发送info命令，用于发现slave节点并确认主从关系。
2. 2s一次对其他sentinal发送节点信息（发布订阅的方式，信息为主观下线）和自身信息（同步最新配置）。
3. 1s一次对sentinal和节点发送ping命令确认存活。

## 集群
