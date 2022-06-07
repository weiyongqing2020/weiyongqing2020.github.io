# redis简介
redis是单线程的：redis的性能瓶颈不在于CPU，而在于内存和网络带宽；单线程可以避免上下文切换。至于redis的持久化操作，是通过fork一个子进程去完成的。
# redis的安装
解压

编译

注：先要安装gcc环境，并注意gcc与redis的版本，redis6.0以上的版本有新特性，需要配合4.9以上版本的gcc，不然会编译失败

# redis 基础知识
共有16个数据库，默认连接的是db0

常用命令：
- select:切换数据库
- keys *:查找当前数据库的所有key
- flushall:清空所有数据库的数据
- flushdb:清空当前数据库的数据
- expire [key]:给key设置过期时间
- ttl [key]:查看key的过期时间 

# 常用的数据类型
## String
常用命令：
- incr [key] :key加1
- decr [key]：key减1
- incrby [key] [num]：key加num
- decrby [key] [num]：key减num
- setnx [key]  [value]：如果不存在即创建
- setex [key]  [time] [value] ：创建并设置key的存活时间
- getset [key] [value]：设置新值，返回之前的值
- mset [key1] [value1]  [key2] [value2] :设置多对
- mget [key1]  [key2]  :查询多对


由于redis并没有一个明确的类型来表示整型数据，所以这个操作是一个字符串操作。执行这个操作的时候，key对应存储的字符串被解析为10进制的64位有符号整型数据。

## List
List是基于链表实现的，可以在两个端点插入和取出数据。

Redis中的列表list，在版本3.2之前，列表底层的编码是ziplist和linkedlist实现的，但是在版本3.2之后，重新引入 quicklist，列表的底层都由quicklist实现。


插入数据的命令：lpush,rpush,  （push可以赋值多个）  lset(只能对已存在的list中已存在的元素赋值)
```
127.0.0.1:6379> lpush mylist wei 
(integer) 1
127.0.0.1:6379> lpush mylist wei1 wei2
(integer) 3
127.0.0.1:6379> rpush mylist wei wei1 wei2
(integer) 6
127.0.0.1:6379> lset mylist 0 hello
OK
```

取出数据的命令：lpop ,rpop
```
127.0.0.1:6379> lpop mylist
"wei2"
127.0.0.1:6379> rpop mylist
"wei2"
```

获取元素的命令：lrange
```
127.0.0.1:6379> lrange mylist 0 2
1) "wei2"
2) "wei1"
3) "wei"
127.0.0.1:6379> lrange mylist 0 -1
1) "wei2"
2) "wei1"
3) "wei"
4) "wei"
5) "wei1"
6) "wei2"
```

获取元素的个数：
```
127.0.0.1:6379> llen mylist 
(integer) 4
```
删除指定元素：第二个参数删除该元素的个数，数量可正负，表示从左或从右删除;如果数量为0，表示删除全部与给定值相符的项
```
127.0.0.1:6379> lrem mylist 0 wei
(integer) 2
```
ltrim：保留指定索引区间的元素，格式是：ltrim list的key 起始索引 结束索引
```
127.0.0.1:6379> ltrim mylist 0 0
OK
```

阻塞命令：例如blpop、brpop：弹出值，格式是：blpop list的key值 过期时间。（key可以是多个，如果没有值，会一直等到有值，直到过期）

## Hash
hash结构常用来存储对象

添加与获取命令：  hget,hset,hmget,hmset，hsetnx
```
127.0.0.1:6379> hset boy age 10 name wei
(integer) 2
127.0.0.1:6379> hmget boy age name
1) "10"
2) "wei"
127.0.0.1:6379> hmset biy age 20 name zhang
OK
```
查询命令：hexists(判断是否存在)，hlen(获取该hash的字段个数),hkeys(获取所有key),hvals(获取所有value）
```
127.0.0.1:6379> hexists boy name
(integer) 1
127.0.0.1:6379> hlen boy
(integer) 2
```
hincrby [key] [field] [num]:将field的值与num相加
```
127.0.0.1:6379> hincrby boy age 10
(integer) 30
127.0.0.1:6379> hincrby boy age -10
(integer) 20
```

## Set
无序不可重复

添加删除命令：sadd, srem (删除指定的元素)，spop(随机删除一个元素)
```
127.0.0.1:6379> sadd myset wei zhang li
(integer) 3
127.0.0.1:6379> spop myset 
"wei"
127.0.0.1:6379> srem myset zhang
(integer) 1
```
查询命令：smembers(查询所有元素) scard(查询元素的个数) sismember(判断是否存在该元素)
```
127.0.0.1:6379> scard myset
(integer) 1
127.0.0.1:6379> smembers myset
1) "li"
127.0.0.1:6379> sismember myset li
(integer) 1
```
集合操作：
sdiff: 返回第一个集合与其他集合之间的差异
sunion: 返回集合的并集
sinter:返回集合的交集
```
127.0.0.1:6379> smembers myset
1) "zhang"
2) "li"
3) "wei"
127.0.0.1:6379> smembers myset2
1) "li"
2) "zhao"
3) "wei"
127.0.0.1:6379> sinter myset myset2
1) "li"
2) "wei"
127.0.0.1:6379> sdiff myset myset2
1) "zhang"
127.0.0.1:6379> sunion myset myset2
1) "li"
2) "zhang"
3) "zhao"
4) "wei"
```

## SortSet
有序不可重复集合

常用命令：zadd(添加) ,zrange(查找), zrangebyscore ((查找结果带上score)，zrem (删除一个或多个元素)
```
127.0.0.1:6379> zadd myset 1000 wei
(integer) 1
127.0.0.1:6379> zadd myset 2000 li
(integer) 1
127.0.0.1:6379> zadd myset 500 zhang
(integer) 1
127.0.0.1:6379> zrange myset 0 -1
1) "zhang"
2) "wei"
3) "li"
127.0.0.1:6379> zrangebyscore myset 0 19999
1) "zhang"
2) "wei"
3) "li"
127.0.0.1:6379> zrangebyscore myset 0 1999
1) "zhang"
2) "wei"
127.0.0.1:6379> zrangebyscore myset 0 1999 withscores
1) "zhang"
2) "500"
3) "wei"
4) "1000"
127.0.0.1:6379> zrem myset zhang
(integer) 1
```

# redis的事务
**redis的事务不保证原子性**

multi 开启事务  
...     要执行的命令  
exec/discard 执行事务/放弃事务  
```
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1 
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> exec
1) OK
2) OK
3) "v2"
```


两种事务异常：
- 编译型异常：命令语法有错误，所有的命令都不会执行
- 运行时异常：出错的命令抛出异常，其他的命令正常执行
```
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set k1 v1
QUEUED
127.0.0.1:6379> incr k1
QUEUED
127.0.0.1:6379> set k2 v2
QUEUED
127.0.0.1:6379> get k2
QUEUED
127.0.0.1:6379> exec
1) OK
2) (error) ERR value is not an integer or out of range
3) OK
4) "v2"
```

redis实现乐观锁：
执行事务之前先监控（watch）key，如果key发生了改变，则事务执行失败
```
127.0.0.1:6379> watch money
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> incrby money 20
QUEUED
127.0.0.1:6379> exec
(nil)
```

# springboot整合redis
为什么要用lettuce替代jedis：
jedis采用直连的方式，多线程操作不安全，想要避免不安全就要使用jedis pool。BIO。
lettuce底层采用netty，实例在多线程中共享，不存在线程不安全，减少线程。NIO

可以通过自定义RedisTemplate的方式来更换序列化方式等。

# redis的配置文件
网络配置：
```
bind: 127.0.0.1
protecteed-mode: yes 保护模式，默认开启
port: 6379
```
日志配置：
```
# 日志
#Specify the server verbosity level.
# This can be one of:
# debug (a lot of information, useful for development/testing)
# verbose (many rarely useful info, but not a mess like the debug level)
# notice (moderately verbose, what you want in production probably) 生产环境
# warning (only very important / critical messages are logged)
loglevel notice
logfile "" # 日志的文件位置名
```

客户端数量：
```
maxclients 10000 # 设置能连接上redis的最大客户端的数量
```

内存达到上限之后的处理策略：
1. volatile-lru：只对设置了过期时间的key进行LRU（默认值）
2. allkeys-lru ： 删除lru算法的key
3. volatile-random：随机删除即将过期key
4. allkeys-random：随机删除
5. volatile-ttl ： 删除即将过期的
6. noeviction ： 永不过期，返回错误
```
maxmemory <bytes> # redis 配置最大的内存容量
maxmemory-policy noeviction # 内存到达上限之后的处理策略
```

# redis的持久化
## rdb
将内存中的数据集快照写入数据库 ，在恢复时候，直接读取快照文件，进行数据的恢复 。默认情况下， Redis 将数据库快照保存在名字为 dump.rdb的二进制文件中。文件名可以在配置文件中进行自定义。

Redis会单独创建（fork）一个子进程来进行持久化，会先将数据写入到一个临时文件中，待持久化过程都结束了，再用这个临时文件替换上次持久化好的文件。整个过程中，主进程是不进行任何IO操作的。

rdb的触发机制：
- 配置文件中的save条件满足时，触发rdb持久化  
- flushall和关闭redis服务的时候也会触发rdb  
- 执行save/bgsave命令

save与bgsave的区别：  
使用 save 命令，会立刻对当前内存中的数据进行持久化 ,但是会阻塞，也就是不接受其他操作了；由于 save 命令是同步命令，会占用Redis的主进程。若Redis数据非常多时，save命令执行速度会非常慢，阻塞所有客户端的请求。
bgsave 是异步进行，进行持久化的时候，redis 还可以将继续响应客户端请求；
|命令| save| bgsave|
|----| ----| ----|
|IO类型| 同步 |异步|
|阻塞？| 是| 是（阻塞发生在fock()，通常非常快）|
|复杂度| O(n)| O(n)|
|优点| 不会消耗额外的内存| 不阻塞客户端命令|
|缺点| 阻塞客户端命令 |需要fock子进程，消耗内存|

只需要将rdb文件放在我们redis启动目录就可以，redis启动的时候会自动检查dump.rdb 恢复其中的数据。

优缺点
- 优点：
    - 适合大规模的数据恢复，相比aof文件更小，效率更高
    - 适合对数据的完整性要求不高的场景
- 缺点：
    - 需要一定的时间间隔进行操作，如果redis意外宕机了，这个最后一次修改之后的数据就没有了。
    - fork进程的时候，会占用一定的内容空间。

## aof
Append Only File  
将我们所有写操作的命令都记录下来，history，恢复的时候就把这个文件全部再执行一遍.

aof默认是不开启的，我们需要手动配置，然后重启redis。


- appendfsync always # 每次修改都会sync 消耗性能
- appendfsync everysec # 每秒执行一次 sync 可能会丢失这一秒的数据
- appendfsync no # 不执行 sync ,这时候操作系统自己同步数据，速度最快

同时开启两种持久化方式：
在这种情况下，当redis重启的时候会优先载入AOF文件来恢复原始的数据，因为在通常情况下AOF文件保存的数据集要比RDB文件保存的数据集要完整。
RDB 的数据不实时，同时使用两者时服务器重启也只会找AOF文件，那要不要只使用AOF呢？建议不要，因为RDB更适合用于备份数据库，快速重启，而且不会有AOF可能潜在的Bug，留着作为一个万一的手段。

aof的优缺点：
- 优点：每一次修改都会同步，文件的完整性会更加好
如果设置每秒同步一次，可能会丢失一秒的数据
- 缺点：相对于数据文件来说，aof远远大于rdb，修复速度比rdb慢。Aof运行效率也要比rdb慢，所以我们redis默认的配置就是rdb持久化。

# redis的发布订阅
Redis 发布订阅(pub/sub)是一种消息通信模式：发送者(pub)发送消息，订阅者(sub)接收消息。

下图展示了频道 channel1 ， 以及订阅这个频道的三个客户端 —— client2 、 client5 和 client1 之间的关系：
![](https://notebook-pictures.obs.cn-north-4.myhuaweicloud.com/051321553483.png)


当有新消息通过 PUBLISH 命令发送给频道 channel1 时， 这个消息就会被发送给订阅它的三个客户端：

发布订阅的常用命令：
- PSUBSCRIBE pattern [pattern..] 订阅一个或多个符合给定模式的频道。
- PUNSUBSCRIBE pattern [pattern..] 退订一个或多个符合给定模式的频道。
- PUBSUB subcommand [argument[argument]] 查看订阅与发布系统状态。
- PUBLISH channel message 向指定频道发布消息
- SUBSCRIBE channel [channel..] 订阅给定的一个或多个频道。
- UNSUBSCRIBE channel [channel..] 退订一个或多个频道
- 
代码示例:
```
------------订阅端----------------------
127.0.0.1:6379> SUBSCRIBE sakura # 订阅sakura频道
Reading messages... (press Ctrl-C to quit) # 等待接收消息
1) "subscribe" # 订阅成功的消息
2) "sakura"
3) (integer) 1
1) "message" # 接收到来自sakura频道的消息 "hello world"
2) "sakura"
3) "hello world"
1) "message" # 接收到来自sakura频道的消息 "hello i am sakura"
2) "sakura"
3) "hello i am sakura"
--------------消息发布端-------------------
127.0.0.1:6379> PUBLISH sakura "hello world" # 发布消息到sakura频道
(integer) 1
127.0.0.1:6379> PUBLISH sakura "hello i am sakura" # 发布消息
(integer) 1
-----------------查看活跃的频道------------
127.0.0.1:6379> PUBSUB channels
"sakura"
```

**原理:**  
Redis是使用C实现的，通过分析 Redis 源码里的 pubsub.c 文件，了解发布和订阅机制的底层实现，籍此加深对 Redis 的理解。
Redis 通过 PUBLISH 、SUBSCRIBE 和 PSUBSCRIBE 等命令实现发布和订阅功能。
每个 Redis 服务器进程都维持着一个表示服务器状态的 redis.h/redisServer 结构， 结构的 pubsub_channels 属性是一个字典， 这个字典就用于保存订阅频道的信息，其中，字典的键为正在被订阅的频道， 而字典的值则是一个链表， 链表中保存了所有订阅这个频道的客户端。

客户端订阅，就被链接到对应频道的链表的尾部，退订则就是将客户端节点从链表中移除。
缺点
如果一个客户端订阅了频道，但自己读取消息的速度却不够快的话，那么不断积压的消息会使redis输出缓冲区的体积变得越来越大，这可能使得redis本身的速度变慢，甚至直接崩溃。
这和数据传输可靠性有关，如果在订阅方断线，那么他将会丢失所有在短线期间发布者发布的消息。
应用
消息订阅：公众号订阅，微博关注等等（起始更多是使用消息队列来进行实现）
多人在线聊天室。
稍微复杂的场景，我们就会使用消息中间件MQ处理。

# 主从同步与集群
主从复制，是指将一台Redis服务器的数据，复制到其他的Redis服务器。前者称为主节点（Master）,后者称为从节点（Slave）， 数据的复制是单向的！只能由主节点复制到从节点，从节点不能进行写操作。

默认情况下，每台Redis服务器都是主节点，一个主节点可以有0个或者多个从节点，但每个从节点只能由一个主节点。

**作用**:
- 数据冗余：主从复制实现了数据的热备份，是持久化之外的一种数据冗余的方式。
- 故障恢复：当主节点故障时，从节点可以暂时替代主节点提供服务，是一种服务冗余的方式
- 负载均衡：在主从复制的基础上，配合读写分离，由主节点进行写操作，从节点进行读操作，分担服务器的负载；尤其是在多读少写的场景下，通过多个从节点分担负载，提高并发量。
- 高可用基石：主从复制还是哨兵和集群能够实施的基础。
- 


主从配置步骤：
主服务器无需做配置
配置从机所属主服务器ip和端口
![](https://notebook-pictures.obs.cn-north-4.myhuaweicloud.com/70.jpeg)

配置从机所属主服务器的密码（要将密码设置尽量复杂）
![](https://notebook-pictures.obs.cn-north-4.myhuaweicloud.com/71.jpeg)

需要注意的是，从服务器通常是只读，所以要配置只读（默认是只读，不要更改即可）
![](https://notebook-pictures.obs.cn-north-4.myhuaweicloud.com/72.jpeg)




**复制原理**：  
Slave 启动成功连接到 master 后会发送一个sync同步命令
Master 接到命令，启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，master将传送整个数据文件到slave，并完成一次完全同步。
全量复制：而slave服务在接收到数据库文件数据后，将其存盘并加载到内存中。
增量复制：Master 继续将新的所有收集到的修改命令依次传给slave，完成同步。
但是只要是重新连接master，一次完全同步（全量复制）将被自动执行！ 我们的数据一定可以在从机中看到！

## 哨兵模式
主从切换技术的方法是：当主服务器宕机后，需要手动把一台从服务器切换为主服务器，这就需要人工干预，费事费力，还会造成一段时间内服务不可用。这不是一种推荐的方式，更多时候，我们优先考虑哨兵模式。Redis从2.8开始正式提供了Sentinel（哨兵） 架构来解决这个问题。能够后台监控主机是否故障，如果故障了根据投票数自动将从库转换为主库。

哨兵模式是一种特殊的模式，首先Redis提供了哨兵的命令，哨兵是一个独立的进程，作为进程，它会独立运行。其原理是哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例。  

**哨兵的作用**：
通过发送命令，让Redis服务器返回监控其运行状态，包括主服务器和从服务器。
当哨兵监测到master宕机，会自动将slave切换成master，然后通过发布订阅模式通知其他的从服务器，修改配置文件，让它们切换主机。
然而一个哨兵进程对Redis服务器进行监控，可能会出现问题，为此，我们可以使用多个哨兵进行监控。各个哨兵之间还会进行监控，这样就形成了多哨兵模式。

**哨兵的作用机制**：
假设主服务器宕机，哨兵1先检测到这个结果，系统并不会马上进行failover过程，仅仅是哨兵1主观的认为主服务器不可用，这个现象成为主观下线。当后面的哨兵也检测到主服务器不可用，并且数量达到一定值时，那么哨兵之间就会进行一次投票，投票的结果由一个哨兵发起，进行failover(故障转移)操作。切换成功后，就会通过发布订阅模式，让各个哨兵把自己监控的从服务器实现切换主机，这个过程称为客观下线。

**配置**
```
配置哨兵配置文件 sentinel.conf
# sentinel monitor 被监控的名称 host port 1
sentinel monitor myredis 127.0.0.1 6379 1
后面的这个数字1，代表主机挂了，slave投票看让谁接替成为主机，票数最多的，就会成为主机。
# Example sentinel.conf
哨兵sentinel实例运行的端口 默认26379
port 26379
哨兵sentinel的工作目录
dir /tmp
哨兵sentinel监控的redis主节点的 ip port
master-name  可以自己命名的主节点名字 只能由字母A-z、数字0-9 、这三个字符".-_"组成。
quorum 当这些quorum个数sentinel哨兵认为master主节点失联 那么这时 客观上认为主节点失联了
sentinel monitor <master-name> <ip> <redis-port> <quorum>
sentinel monitor mymaster 127.0.0.1 6379 1
当在Redis实例中开启了requirepass foobared 授权密码 这样所有连接Redis实例的客户端都要提供密码
设置哨兵sentinel 连接主从的密码 注意必须为主从设置一样的验证密码
sentinel auth-pass <master-name> <password>
sentinel auth-pass mymaster MySUPER--secret-0123passw0rd
指定多少毫秒之后 主节点没有应答哨兵sentinel 此时 哨兵主观上认为主节点下线 默认30秒
sentinel down-after-milliseconds <master-name> <milliseconds>
sentinel down-after-milliseconds mymaster 30000
这个配置项指定了在发生failover主备切换时最多可以有多少个slave同时对新的master进行 同步，
这个数字越小，完成failover所需的时间就越长，
但是如果这个数字越大，就意味着越 多的slave因为replication而不可用。
可以通过将这个值设为 1 来保证每次只有一个slave 处于不能处理命令请求的状态。
sentinel parallel-syncs <master-name> <numslaves>
sentinel parallel-syncs mymaster 1
故障转移的超时时间 failover-timeout 可以用在以下这些方面：
1. 同一个sentinel对同一个master两次failover之间的间隔时间。
2. 当一个slave从一个错误的master那里同步数据开始计算时间。直到slave被纠正为向正确的master那里同步数据时。
3.当想要取消一个正在进行的failover所需要的时间。
4.当进行failover时，配置所有slaves指向新的master所需的最大时间。不过，即使过了这个超时，slaves依然会被正确配置为指向master，但是就不按parallel-syncs所配置的规则来了
默认三分钟
sentinel failover-timeout <master-name> <milliseconds>
sentinel failover-timeout mymaster 180000
SCRIPTS EXECUTION
配置当某一事件发生时所需要执行的脚本，可以通过脚本来通知管理员，例如当系统运行不正常时发邮件通知相关人员。
对于脚本的运行结果有以下规则：
若脚本执行后返回1，那么该脚本稍后将会被再次执行，重复次数目前默认为10
若脚本执行后返回2，或者比2更高的一个返回值，脚本将不会重复执行。
如果脚本在执行过程中由于收到系统中断信号被终止了，则同返回值为1时的行为相同。
一个脚本的最大执行时间为60s，如果超过这个时间，脚本将会被一个SIGKILL信号终止，之后重新执行。
通知型脚本:当sentinel有任何警告级别的事件发生时（比如说redis实例的主观失效和客观失效等等），将会去调用这个脚本，
这时这个脚本应该通过邮件，SMS等方式去通知系统管理员关于系统不正常运行的信息。调用该脚本时，将传给脚本两个参数，
一个是事件的类型，
一个是事件的描述。
如果sentinel.conf配置文件中配置了这个脚本路径，那么必须保证这个脚本存在于这个路径，并且是可执行的，否则sentinel无法正常启动成功。
通知脚本
sentinel notification-script <master-name> <script-path>
sentinel notification-script mymaster /var/redis/notify.sh
客户端重新配置主节点参数脚本
当一个master由于failover而发生改变时，这个脚本将会被调用，通知相关的客户端关于master地址已经发生改变的信息。
以下参数将会在调用脚本时传给脚本:
<master-name> <role> <state> <from-ip> <from-port> <to-ip> <to-port>
目前<state>总是“failover”,
<role>是“leader”或者“observer”中的一个。
参数 from-ip, from-port, to-ip, to-port是用来和旧的master和新的master(即旧的slave)通信的
这个脚本应该是通用的，能被多次调用，不是针对性的。
sentinel client-reconfig-script <master-name> <script-path>
sentinel client-reconfig-script mymaster /var/redis/reconfig.sh
```



启动哨兵 redis-sentinel ./sentinal.conf
![](https://notebook-pictures.obs.cn-north-4.myhuaweicloud.com/75png.png)
此时哨兵监视着我们的主机6379，当我们断开主机后：
![](https://notebook-pictures.obs.cn-north-4.myhuaweicloud.com/76.png)



# 缓存穿透与雪崩
## 缓存穿透
**定义**：在默认情况下，用户请求数据时，会先在缓存(Redis)中查找，若没找到即缓存未命中，再在数据库中进行查找，数量少可能问题不大，可是一旦大量的请求数据（例如秒杀场景）缓存都没有命中的话，就会全部转移到数据库上，造成数据库极大的压力，就有可能导致数据库崩溃。网络安全中也有人恶意使用这种手段进行攻击被称为洪水攻击。

**解决方案**：
布隆过滤器：
对所有可能查询的参数以Hash的形式存储，以便快速确定是否存在这个值，在控制层先进行拦截校验，校验不通过直接打回，减轻了存储系统的压力。
![](https://notebook-pictures.obs.cn-north-4.myhuaweicloud.com/13215824722.jpeg)
缓存空对象：
一次请求若在缓存和数据库中都没找到，就在缓存中方一个空对象用于处理后续这个请求。这样做有一个缺陷：存储空对象也需要空间，大量的空对象会耗费一定的空间，存储效率并不高。解决这个缺陷的方式就是设置较短过期时间。即使对空值设置了过期时间，还是会存在缓存层和存储层的数据会有一段时间窗口的不一致，这对于需要保持一致性的业务会有影响。
![](https://notebook-pictures.obs.cn-north-4.myhuaweicloud.com/13215836317.jpeg)
## 缓存击穿
**定义**：相较于缓存穿透，缓存击穿的目的性更强，一个存在的key，在缓存过期的一刻，同时有大量的请求，这些请求都会击穿到DB，造成瞬时DB请求量大、压力骤增。这就是缓存被击穿，只是针对其中某个key的缓存不可用而导致击穿，但是其他的key依然可以使用缓存响应。
​ 比如热搜排行上，一个热点新闻被同时大量访问就可能导致缓存击穿。

**解决方案**：
1. 设置热点数据永不过期
这样就不会出现热点数据过期的情况，但是当Redis内存空间满的时候也会清理部分数据，而且此种方案会占用空间，一旦热点数据多了起来，就会占用部分空间。
2. 加互斥锁(分布式锁)
在访问key之前，采用SETNX（set if not exists）来设置另一个短期key来锁住当前key的访问，访问结束再删除该短期key。保证同时刻只有一个线程访问。这样对锁的要求就十分高。

## 缓存雪崩
**定义**：缓存雪崩可能有两种情况：
1. 大量的key设置了相同的过期时间，导致在缓存在同一时刻全部失效，造成瞬时DB请求量大、压力骤增，引起雪崩。缓存雪崩，是指在某一个时间段，缓存集中过期失效。Redis 宕机！产生雪崩的原因之一，比如在写本文的时候，马上就要到双十二零点，很快就会迎来一波抢购，这波商品时间比较集中的放入了缓存，假设缓存一个小时。那么到了凌晨一点钟的时候，这批商品的缓存就都过期了。而对这批商品的访问查询，都落到了数据库上，对于数据库而言，就会产生周期性的压力波峰。于是所有的请求都会达到存储层，存储层的调用量会暴增，造成存储层也会挂掉的情况。
2. 其实集中过期，倒不是非常致命，比较致命的缓存雪崩，是缓存服务器某个节点宕机或断网。因为自然形成的缓存雪崩，一定是在某个时间段集中创建缓存，这个时候，数据库也是可以顶住压力的。无非就是对数据库产生周期性的压力而已。而缓存服务节点的宕机，对数据库服务器造成的压力是不可预知的，很有可能瞬间就把数据库压垮。
![](https://notebook-pictures.obs.cn-north-4.myhuaweicloud.com/13215850428.jpeg)
**解决方案**
- redis高可用：这个思想的含义是，既然redis有可能挂掉，那我多增设几台redis，这样一台挂掉之后其他的还可以继续工作，其实就是搭建的集群。
- 限流降级：这个解决方案的思想是，在缓存失效后，通过加锁或者队列来控制读数据库写缓存的线程数量。比如对某个key只允许一个线程查询数据和写缓存，其他线程等待。
- 数据预热：数据加热的含义就是在正式部署之前，我先把可能的数据先预先访问一遍，这样部分可能大量访问的数据就会加载到缓存中。在即将发生大并发访问前手动触发加载缓存不同的key，设置不同的过期时间，让缓存失效的时间点尽量均匀。









